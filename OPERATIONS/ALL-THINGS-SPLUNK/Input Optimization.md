# Splunk Universal Forwarder – Input Optimization Lab

## Objective
Configure the Splunk Universal Forwarder to reduce unnecessary data ingest, eliminate duplicate collection, enforce least privilege, and improve overall efficiency.

---

## 1. Disable Low-Value / High-Volume Inputs

**Inputs Disabled:** `iostat`, `nfsiostat`, `ps`, `top`, `protocol`, `lsof`

These scripted inputs generate extremely high log volumes with minimal investigative value. Disabling them reduces ingest noise and license consumption without impacting detection capability.

| Input | Reason for Disabling |
|---|---|
| `iostat` | Disk I/O stats — useful only for niche performance troubleshooting |
| `nfsiostat` | NFS mount stats — not relevant to security monitoring |
| `ps` | Generates 1,440 process snapshots/day — rarely queried |
| `top` | Redundant with `ps`, high volume, low signal |
| `protocol` | Network protocol stats — low value for SOC use cases |
| `lsof` | Extremely verbose open file/socket listing |

---

## 2. Disable "time" Input

The `time` input was designed for environments running `ntpd`. This environment uses **chronyd**, the modern time sync daemon standard in RHEL/CentOS 7+. The input is incompatible and returns unreliable data, so it has been disabled.

---

## 3. Disable "rlog" Input

The `rlog` input duplicates events already collected directly from `/var/log/audit/audit.log` via a monitor stash. Specifically, it runs in the background and gathers logs from sources like:
```
/var/log/messages
/var/log/syslog
System event logs depending on the OS.
```
Keeping both enabled results in **double ingestion** of the same events, wasting license capacity with no added value.

---

## 4. Disable "bash_history" and "sshdChecker"

Both inputs require the Splunk Forwarder to run as **root** in order to access protected system files. This violates the principle of **least privilege** — if the forwarder were compromised, an attacker would gain full root access to the host. These inputs have been disabled to reduce attack surface.

If an attacker compromises the Splunk Forwarder (through a vulnerability in Splunk itself, a malicious config file, or a tampered scripted input), they can:

- Replace or modify a Splunk script with a malicious one
- Splunk executes it automatically on its polling schedule
- Since Splunk is root, that malicious script runs as root
- The attacker now has full control of the system
  
```
For example, a tampered script could:
# Add attacker as root user
echo "attacker:x:0:0:root:/root:/bin/bash" >> /etc/passwd
```
---

## 5. Disable Collection from "/Library/Logs"

This path is specific to **macOS** systems. Since no Macs exist in this environment, the path returns no data and only introduces unnecessary errors in the forwarder configuration.

---

## 6. Disable Collection from "/etc"

The `/etc` directory contains system configuration files such as `passwd`, `hosts`, and `fstab` — not time-stamped log files. Splunk's monitor input requires timestamp-based logs to index data properly. Collecting from `/etc` produces malformed or unindexable events.

What Splunk Expects
Splunk's monitor input is designed to read log files — files where each line has a timestamp attached to it, like:
```
May 12 10:23:41 sshd[1234]: Accepted password for user from 192.168.1.5
May 12 10:23:42 systemd[1]: Started Session 5 of user admin.
```
What /etc Files Look Like
Files in /etc have no timestamps on each line. They're just static configuration, like /etc/passwd:
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
user1:x:1001:1001::/home/user1:/bin/bash
```
or /etc/hosts:
```
127.0.0.1   localhost
192.168.1.10  webserver
```
---

## 7. Reduce Collection Frequency — vmstat & df

**New Frequency:** 1 poll / hour (down from 1 poll / minute)

`vmstat` (virtual memory stats) and `df` (disk space) change slowly over time. Polling every minute generates **1,440 data points per host per day**. Reducing to hourly collection (24 points/day) retains meaningful trend visibility while significantly cutting ingest volume.

```
60 checks/hour × 24 hours = 1,440 data points per day per host

1 poll/hour (reduced)

1 check/hour × 24 hours = 24 data points per day per host
(1,440 - 24) / 1,440 × 100

= 1,416 / 1,440 × 100

= 0.9833 × 100

= 98.3%

Eliminates 1,416 out of 1,440 data points per day
```
---

## 8. Reduce Collection Frequency — package & hardware

**New Frequency:** 1 poll / day

Installed package lists and hardware inventory are largely static. Daily collection is sufficient to detect changes (e.g., unauthorized software installation) without the overhead of frequent polling.

---

## 9. Add Regex Exclusion for Rotated Backup Logs

**Regular Expression:** `\.\d+$`

When Linux rotates logs, it renames them by appending a number:
syslog.1
syslog.2
syslog.3

```
\. A literal dot. The backslash is an escape character because in regex, a plain . means "any character" — so \. tells it to look for an actual period.

- \d+

- \d means any digit (0-9)
- + means one or more of them

- So \d+ matches 1, 2, 23, 100, etc.

$
Means end of the string/filename. The pattern has to match at the very end.

Put Together: \.\d+$

"Match anything that ends in a dot followed by one or more numbers"

So it catches:
syslog.1       ✓
syslog.2       ✓
syslog.23      ✓
auth.log.1     ✓
auth.log.100   ✓
```
Without an exclusion, Splunk attempts to ingest these archived files as new data, causing **duplicate events**. The regex `\.\d+$` matches any filename ending in a dot followed by one or more digits, instructing Splunk to skip rotated log files entirely.

---

## Summary

| Change | Impact |
|---|---|
| Disabled 6 high-volume inputs | Reduced ingest noise and license usage |
| Disabled `time` input | Removed incompatible/unreliable data |
| Disabled `rlog` | Eliminated duplicate audit log ingestion |
| Disabled `bash_history` & `sshdChecker` | Enforced least privilege on forwarder |
| Disabled `/Library/Logs` collection | Removed irrelevant macOS path |
| Disabled `/etc` collection | Removed non-log configuration directory |
| Reduced `vmstat` & `df` to 1/hour | Cut polling volume by ~98% |
| Reduced `package` & `hardware` to 1/day | Minimized static data overhead |
| Added rotated log exclusion regex | Prevented duplicate event ingestion |
