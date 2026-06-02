 Brute Force Attack Triage Playbook

**Tactic:** Credential Access  
**MITRE ATT&CK:** T1110 — sub-techniques T1110.001 (Password Guessing), T1110.003 (Password Spraying), T1110.004 (Credential Stuffing)  
**Severity:** High (escalates to Critical if successful logon confirmed)  
**SLA:** Acknowledge within 15 min · Contain within 45 min if successful logon detected  
**Last updated:** 2025-06-03  

---

## 1. Alert context

This playbook covers alerts fired by the following detections:

| Detection | File | Trigger |
|---|---|---|
| Account lockout spike | `detections/credential_access/T1110.001_brute_force/query.spl` | EID 4740 — 5+ lockouts in 10 min |
| Failed logon threshold | `detections/credential_access/T1110.001_brute_force/query.spl` | EID 4625 — 20+ failures from single source |
| Password spraying | `detections/credential_access/T1110.003_password_spraying/query.spl` | EID 4625 — 1 failure across 10+ accounts from single source |
| Successful logon after failures | `detections/credential_access/T1110.001_brute_force/query.spl` | EID 4625 then 4624 from same source within 5 min |

**Why this matters:** A failed brute force is a nuisance. A successful one is a breach. The entire triage priority hinges on whether any attempt succeeded — check this first before anything else.

---

## 2. Immediate triage (0–15 min)

### Step 1 — Determine if any attempt succeeded

This is the single most important question. Run it first.

```spl
index=windows earliest=-1h
    EventCode IN (4625, 4624)
| eval status=if(EventCode=4624, "SUCCESS", "FAILURE")
| stats 
    count(eval(status="FAILURE")) as failures,
    count(eval(status="SUCCESS")) as successes,
    values(TargetUserName) as accounts,
    values(IpAddress) as src_ips
    by IpAddress, status
| where failures > 5
```

If `successes > 0` from the same source IP that generated failures → **treat as active breach, skip to Section 4 containment immediately.**

---

### Step 2 — Classify the attack pattern

The pattern tells you what you are dealing with and what the attacker is after.

```spl
index=windows earliest=-1h
    EventCode=4625
| stats 
    dc(TargetUserName) as unique_accounts,
    dc(IpAddress) as unique_sources,
    count as total_failures
    by IpAddress
| eval pattern=case(
    unique_accounts=1 AND total_failures > 20, "Traditional brute force — single account",
    unique_accounts > 10 AND total_failures < 50, "Password spraying — many accounts, low volume",
    unique_accounts > 1 AND unique_sources > 5, "Credential stuffing — distributed sources",
    true(), "Unknown — investigate manually"
)
| table IpAddress, unique_accounts, unique_sources, total_failures, pattern
| sort -total_failures
```

| Pattern | Attacker goal | Urgency |
|---|---|---|
| Traditional brute force | Compromise one specific account | High |
| Password spraying | Avoid lockout, blend in with noise | Critical — hard to detect, often succeeds |
| Credential stuffing | Validate leaked credential lists | High — implies known passwords |

---

### Step 3 — Identify targeted accounts

```spl
index=windows earliest=-1h
    EventCode=4625
| stats 
    count as failures,
    values(IpAddress) as src_ips,
    dc(IpAddress) as src_count
    by TargetUserName
| sort -failures
| head 20
```

Flag immediately if targeted accounts include:

- Domain Admin or other privileged accounts
- Service accounts
- Accounts belonging to executives or finance staff
- Accounts that were recently created (potential insider threat angle)

---

### Step 4 — Characterize the source

```spl
index=windows earliest=-1h
    EventCode=4625
| stats count by IpAddress, TargetUserName, LogonType, WorkstationName
| sort -count
```

| `LogonType` | Meaning | Implication |
|---|---|---|
| 2 | Interactive (console) | Attacker has physical or RDP access |
| 3 | Network | SMB, file share — common for spraying |
| 7 | Unlock | Screensaver unlock attempts |
| 8 | NetworkCleartext | Cleartext password — legacy protocol abuse |
| 10 | RemoteInteractive | RDP |

Check the source IP:
- Is it internal? → Compromised internal host or malicious insider
- Is it a known VPN range? → Potentially legitimate but investigate
- Is it external? → Block at perimeter firewall immediately
- Is it a Tor exit node or hosting provider range? → High confidence malicious

---

## 3. Scope expansion (15–30 min)

### Check for spraying across multiple systems

```spl
index=windows earliest=-2h
    EventCode=4625
| stats 
    dc(ComputerName) as systems_targeted,
    dc(TargetUserName) as accounts_targeted,
    count as failures
    by IpAddress
| where systems_targeted > 3
| sort -systems_targeted
```

Spraying across many systems simultaneously is a hallmark of an automated tool (Spray, Ruler, MSOLSpray, etc.). More than 5 systems from one source in under an hour is almost certainly automated.

### Check for spraying against cloud / federated services

If your environment uses Azure AD, M365, or ADFS, brute force may be hitting cloud endpoints rather than on-prem DC, and EID 4625 will be absent or sparse. Check:

```spl
index=o365 OR index=azuread earliest=-2h
    Operation IN ("UserLoginFailed", "UserLoggedIn")
| stats 
    count(eval(Operation="UserLoginFailed")) as failures,
    count(eval(Operation="UserLoggedIn")) as successes
    by ClientIP, UserId
| where failures > 10
| sort -failures
```

### Identify all systems the source IP touched

```spl
index=windows earliest=-4h
    IpAddress="<ATTACKER_IP>"
| stats count by ComputerName, EventCode, TargetUserName
| sort ComputerName
```

This tells you whether the attacker recon'd other systems or only targeted auth endpoints.

---

## 4. Containment actions

### If NO successful logon confirmed

| # | Action | Who | Notes |
|---|---|---|---|
| 1 | Block source IP at perimeter firewall | Network / SOC | If external. Document rule with case ID |
| 2 | Block source IP in Windows Firewall via GPO | SOC / IT | Belt-and-suspenders for internal reach |
| 3 | Enable fine-grained password policy if not set | IAM | Ensure lockout threshold is ≤ 10 attempts |
| 4 | Alert owners of targeted accounts | SOC | Advise to change password as precaution |
| 5 | Monitor for resumed attempts from new IPs | SOC | Attacker may rotate — create a watchlist |

### If successful logon IS confirmed — treat as active breach

| # | Action | Who | Notes |
|---|---|---|---|
| 1 | Disable the compromised account immediately | IAM | Do not just reset — disable to kill active sessions |
| 2 | Revoke all active sessions / tokens | IAM | Azure AD: Revoke-AzureADUserAllRefreshToken |
| 3 | Isolate any host the account authenticated to | SOC | Check EID 4624 — what did they log into? |
| 4 | Block attacker IP at all perimeter points | Network | Firewall, WAF, VPN concentrator |
| 5 | Escalate to incident lead | SOC lead | This is now an active incident |
| 6 | Preserve logs before any remediation | SOC | Export from Splunk, pull evtx files |

---

## 5. Evidence collection

```
[ ] Windows Security event logs — EID 4625, 4624, 4740, 4768, 4771 (last 24h minimum)
[ ] Source IP reputation lookups — VirusTotal, AbuseIPDB, Shodan
[ ] Firewall / proxy logs for source IP (what else did this IP do?)
[ ] VPN authentication logs if source is a VPN range
[ ] M365 / Azure AD sign-in logs if federated auth is in scope
[ ] LDAP query logs if spraying was against LDAP directly (EID 1644 on DC)
[ ] List of all accounts that reached lockout threshold
[ ] Timestamps of first and last observed attempt
```

Store to: `incidents/YYYY-MM-DD_<CASEID>/evidence/`

---

## 6. Forensic analysis

### Determine the password used in successful logon (if applicable)

You cannot recover the password from logs, but you can determine the logon method:

```spl
index=windows earliest=-2h
    EventCode=4624
    IpAddress="<ATTACKER_IP>"
| table _time, TargetUserName, LogonType, AuthenticationPackageName, 
         WorkstationName, IpAddress, IpPort
```

- `AuthenticationPackageName = NTLM` → password was guessed or stolen hash was used (pass-the-hash possible)
- `AuthenticationPackageName = Kerberos` → valid password was used
- `LogonType = 3` + NTLM → strong indicator of pass-the-hash rather than true brute force

### Check what the account did after successful logon

```spl
index=windows earliest=-2h
    SubjectUserName="<COMPROMISED_ACCOUNT>" OR TargetUserName="<COMPROMISED_ACCOUNT>"
    EventCode IN (4688, 4720, 4732, 4728, 5140, 7045)
| table _time, EventCode, SubjectUserName, TargetUserName, 
         CommandLine, ShareName, ServiceName, ComputerName
| sort _time
```

| EventCode | Means the attacker... |
|---|---|
| 4688 | Ran a process — review CommandLine |
| 4720 | Created a new account — persistence |
| 4732 | Added account to local admin group — privilege escalation |
| 4728 | Added account to domain group — privilege escalation |
| 5140 | Accessed a network share — data access or staging |
| 7045 | Installed a service — persistence or lateral tool drop |

Any of the above after a successful logon is a critical escalation indicator.

---

## 7. Escalation criteria

Escalate to Incident Commander if **any** of the following are true:

- [ ] Successful logon confirmed from attacker source
- [ ] Privileged account (Domain Admin, service account) was successfully authenticated
- [ ] Attacker ran processes or accessed shares post-logon
- [ ] Attack is distributed across more than 5 source IPs (botnet / credential stuffing)
- [ ] Attack targets cloud / M365 and on-prem simultaneously
- [ ] Evidence of new account creation or group membership change
- [ ] Attack volume is causing operational impact (mass lockouts disrupting users)

---

## 8. Detection tuning notes

Brute force detections have the highest FP rate of any category. Common legitimate sources to allowlist after verification:

| Source | EventCode | Reason |
|---|---|---|
| Monitoring / vulnerability scanners | 4625 | Scheduled authenticated scans |
| Legacy applications with hardcoded credentials | 4625, 4740 | App using stale password |
| Help desk password reset scripts | 4625 | Testing reset before notifying user |
| Load balancers / proxies | 4625 | Shared source IP masks true client |

For each FP, add an entry to `detections/credential_access/T1110.001_brute_force/detection.yml` under the `false_positives` key rather than hardcoding exclusions in the SPL — keeps the query logic clean and the FP reasoning documented.

---

## 9. Post-incident actions

- [ ] Write incident timeline — `incidents/YYYY-MM-DD_<CASEID>/timeline.md`
- [ ] Submit source IP(s) to AbuseIPDB
- [ ] Review account lockout policy — is threshold appropriate? (recommend ≤ 10 attempts)
- [ ] Review MFA coverage — was the targeted account MFA-enrolled? If not, flag for enforcement
- [ ] Identify how the attacker obtained the username list (public breach data, OSINT, LDAP enumeration)
- [ ] Run a 30-day lookback for the same source IP or attack pattern
- [ ] Update MITRE ATT&CK navigator layer with T1110 sub-technique observed
- [ ] If spraying — check HaveIBeenPwned API for targeted accounts against known breach datasets
- [ ] Schedule lessons-learned within 48h

---

## 10. Reference

| Resource | Link |
|---|---|
| MITRE ATT&CK T1110 | https://attack.mitre.org/techniques/T1110/ |
| Password spraying guidance (CISA) | https://www.cisa.gov/news-events/cybersecurity-advisories/aa21-008a |
| Fine-grained password policies | https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/adac/introduction-to-active-directory-administrative-center-enhancements |
| AbuseIPDB reporting | https://www.abuseipdb.com |
| Detection rules for this tactic | `detections/credential_access/` |

