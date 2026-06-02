***This lab component is an extension of Splunk Enterprise Deployment Lab***

$${{\color{Goldenrod}\Huge{\textsf{  Setting \ Up\ a\ Syslog\ Server \}}}}\$$

$${{\color{Goldenrod}\Huge{\textsf{  Components \}}}}\$$

- Red Hat Enterprise 10 VM - SYSLOG SEVER
- Cluster Manager Configured
- 3 Indexers running Red Hat Enterprise 10

- RHEL (syslog-ng) ──► TCP 5514 ──► IDX1, IDX2, IDX3 
                                        ▲ 
                                   CM manages cluster 

 # syslog-ng → Splunk Indexer Cluster Architecture

## Architecture Overview

```
RHEL (syslog-ng) ──► TCP 5514 ──► IDX1, IDX2, IDX3
                                        ▲
                                   CM manages cluster
```

---

## RHEL — Install & Configure syslog-ng

### Step 1 — Add EPEL repo

```bash
sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```

### Step 2 — Add syslog-ng COPR repo

```bash
curl -o /etc/yum.repos.d/syslog-ng.repo https://copr.fedorainfracloud.org/coprs/czanik/syslog-ng48/repo/epel-9/czanik-syslog-ng48-epel-9.repo
```

### Step 3 — Install syslog-ng

```bash
sudo dnf install syslog-ng -y
```

### Step 4 — Disable rsyslog

```bash
sudo systemctl stop rsyslog
sudo systemctl disable rsyslog
```

### Step 5 — Configure syslog-ng

```bash
sudo bash -c 'cat > /etc/syslog-ng/syslog-ng.conf << "EOF"
@version: 4.8
@include "scl.conf"

source s_local {
    system();
    internal();
    file("/var/log/secure");
    file("/var/log/messages");
};

destination d_idx1 { network("192.168.17.147" transport("tcp") port(5514)); };
destination d_idx2 { network("192.168.17.143" transport("tcp") port(5514)); };
destination d_idx3 { network("192.168.17.150" transport("tcp") port(5514)); };

log {
    source(s_local);
    destination(d_idx1);
    destination(d_idx2);
    destination(d_idx3);
};
EOF'
```

### Step 6 — Disable SELinux

```bash
sudo setenforce 0
```

### Step 7 — Start syslog-ng

```bash
sudo systemctl start syslog-ng
sudo systemctl enable syslog-ng
sudo systemctl status syslog-ng
```

### Step 8 — Verify no errors

```bash
sudo journalctl -xeu syslog-ng --no-pager | tail -20
```

---

## Splunk CM — Configure Inputs

### Step 9 — Edit inputs.conf

```bash
sudo vi /opt/splunk/etc/master-apps/org_all_indexers/local/inputs.conf
```

```ini
[tcp://5514]
index = syslog
sourcetype = syslog
connection_host = dns
disabled = 0
```

### Step 10 — Push bundle to indexers

```bash
/opt/splunk/bin/splunk apply cluster-bundle
```

---

## Each Indexer — IDX1, IDX2, IDX3

### Step 11 — Open firewall

```bash
sudo firewall-cmd --add-port=5514/tcp --permanent
sudo firewall-cmd --reload
```

### Step 12 — Verify Splunk listening

```bash
ss -tnlp | grep 5514
```

Expected output:

```
LISTEN 0 128 0.0.0.0:5514 0.0.0.0:* users:(("splunkd"...))
```

---

## Test

### Step 13 — Send test log from RHEL

```bash
logger "test from rhel"
```
<div align= "center">
<img src="https://github.com/user-attachments/assets/0ee2f338-3c34-402f-985c-1267668c6b39" height="600" width="600" />
</div>

### Step 14 — Search in Splunk

```spl
index=syslog | head 20
```
<div align= "center">
<img src="https://github.com/user-attachments/assets/c8346086-81d8-49ab-8ba1-6616211ee56c" height="500" width="500" />

</div>



