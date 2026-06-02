# ⚙️ Splunk + Atomic Red Team Lab Setup

![image](https://github.com/user-attachments/assets/d6ac83ed-98cd-4072-ab0f-7e3a9a56b65b)


This guide walks you through setting up a lab environment using **Splunk**, **Sysmon**, and **Atomic Red Team** for detection engineering, MITRE ATT&CK mapping, and dashboarding.

---

## 🔧 Level 1: Setup and Installation

### 🧩 Step 1: Install Splunk (Free Version)

- Download Splunk Enterprise: https://www.splunk.com  
- Install on Windows Server 2016 or a Linux VM (Linux preferred for scalability).  
- Use default install options.  
- Access via: `https://localhost:8000`  

---

### 🔩 Step 2: Install Splunk Universal Forwarder on Windows Server 2016

- Purpose: Forward logs to Splunk.
- Download: https://www.splunk.com/en_us/download/universal-forwarder.html
- During setup:
  - Select "Forward Windows Event Logs"
  - Point to Splunk Server IP on port 9997
- Ensure port 9997 is open on Splunk server (check firewall).

---

### 🔐 Step 3: Enable PowerShell Logging

Required for Atomic Red Team visibility.

Enable:

- Module Logging
- Script Block Logging
- Event ID 4104

This can be done in gpedit.msc

Navigate to:

Computer Configuration > Administrative Templates > Windows Components > Windows PowerShell

Turn on the following:

- Turn on Module Logging : Enabled  
- Turn on PowerShell Script Block Logging : Enabled

---

### 🔐 Step 4: Installing Atomic Red Team

Reference: https://github.com/redcanaryco/invoke-atomicredteam/wiki/Installing-Invoke-AtomicRedTeam

Run the following commands in PowerShell as administrator:
```
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
```

```
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);
Install-AtomicRedTeam
```
```
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicsfolder.ps1' -UseBasicParsing); Install-AtomicsFolder
```
---


### 🔐 Step 5: Run Atomic Test - T1136.001 (Create Local User)

Run the Atomic Red Team test for: T1136.001 - Creating a local user

<p align="center">
  <img src="https://github.com/user-attachments/assets/88148c05-2e35-446b-b003-2e05ac3485e1" />
</p>

Once executed, the New-LocalUser action will appear in logs.

<p align="center">
  <img src="https://github.com/user-attachments/assets/6f05fa64-1aaa-4f9e-89aa-860bcbb47fb9" />
</p>

---

### 🔐 Step 7: Finding the ATT&CKs in Splunk

Search query:

index=endpoint New-LocalUser  
| eval mitre_technique_id="T1136.001"  
| eval kill_chain_phase="persistence"  
| table _time, host, CommandLine, User, mitre_technique_id, kill_chain_phase

Save this as a report:

- Add a title
- Schedule it if needed

<p align="center">
  <img src="https://github.com/user-attachments/assets/413111c8-4fe2-43df-9cfb-e3156e60d1e7" />
</p>
### 🔐 Step 8: Mapping Attacks 

- Create a mitre attack csv file
  
| EventID | MITRE Technique | Description                             |
|---------|------------------|-----------------------------------------|
| 4720    | T1136            | Account Created                         |
| 4722    | T1136            | Account Enabled                         |
| 4724    | T1136            | Password Reset Attempt                  |
| 4725    | T1136            | Account Disabled                        |
| 4726    | T1136            | Account Deleted                         |
| 4624    | T1078            | Successful Logon (Valid Account)        |
| 4625    | T1110            | Failed Logon Attempt (Brute Force)      |
| 4688    | T1059            | Command Execution (Process Creation)    |
| 4670    | T1222            | Permissions on Object Changed           |
| 4697    | T1053            | Scheduled Task Created

Run this query
```
index=endpoint
| spath input=_raw path=Event.System.EventID output=EventID
| lookup mitre_eventid_lookup EventID OUTPUT MITRE_Technique Description
| stats count by EventID, MITRE_Technique, Description
```
  1. index=endpoint: Searches your logs in the endpoint index.
  2. spath: Extracts the EventID from the XML log (like 7036) into a field.
  3. lookup: Matches that EventID against the CSV you uploaded (mitre_eventid_lookup.csv) to add:MITRE technique (like T1136)
  4. Description (like Account Created)
  5. stats: Counts how often each EventID + MITRE combo appears.
