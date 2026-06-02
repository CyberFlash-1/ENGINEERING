 $${{\color{Lime}\huge{\textsf{Splunk Enterprise Lab\ }}}}\$$


$${{\color{Yellow}\huge{\textsf{Architecture }}}}\$$
<div align="center">
<img width="800" height="600" alt="deployment" src="https://github.com/user-attachments/assets/08b54ee5-9710-49e9-87f2-44e3835bbd3a" /></a>


</div>
  </br>
  </div>
  


${{\color{Yellow}\large{\textsf{Splunk Distributed Deployment Lab\ }}}}\$

A full multi‑tier Splunk Enterprise lab that mirrors a real‑world distributed architecture. This environment includes clustered indexers, a dedicated Cluster Manager, a Search Head, a combined License Manager / Deployment Server / Monitoring Console, and a Universal Forwarders. The lab demonstrates installation, clustering, index management, data onboarding, search‑time knowledge objects, and distributed monitoring.

---

## 🧩 Architecture Overview

The lab consists of the following Splunk components:

- **License Manager / Deployment Server / Monitoring Console**
- **Cluster Manager (CM)**
- **Indexer Cluster (3 peers)**
- **Search Head (SH)**


Forwarder sends data → Indexer Cluster (via Indexer Discovery) → Search Head for distributed search. The Deployment Server manages UF configurations, and the Monitoring Console runs in distributed mode.

---

## ⚙️ Installation & Core Configuration

### Splunk Enterprise (All Components Except UFs)
- Extract Splunk to `/opt/splunk`
- Assign ownership to the `splunk` user
- Start Splunk and accept the license
- Enable boot‑start with systemd and polkit rules

### Licensing
- Configure each instance as a license *peer*
- Assign the License Manager via port `8089`
- Install the Enterprise license on the LM

### Deployment Server
- Enable DS functionality
- Configure `serverclass.conf` to manage UF apps
- Deploy apps such as `Splunk_TA_nix` and custom monitoring inputs

---

## 🏗️ Indexer Clustering

### Cluster Manager
- Configure clustering in **manager** mode  
  Example:
```
splunk edit cluster-config -mode manager -replication_factor 3 -search_factor 2 -secret <key>
```
- Enable Indexer Discovery for forwarders
- Manage indexes via:
`/opt/splunk/etc/master-apps/_cluster/local/indexes.conf`
- Validate and push cluster bundles

### Indexer Peers
- Join each indexer to the cluster manager
- Open required ports (`8089`, `9997`, replication port)
- Enable receiving on port `9997`

---

## 🔍 Search Head Integration

- Connect SH to the cluster using:
  ```
  splunk edit cluster-config -mode searchhead -master_uri https://<CM>:8089
  ```
  - Restart and validate distributed search
- Install search‑time apps (e.g., Splunk_TA_nix)
- Configure props/transforms for XML and IronPort parsing

---

## 📡 Forwarder Configuration

### Universal Forwarder Installation
- Extract UF to `/opt/splunkforwarder`
- Start and enable the service

### Indexer Discovery
`outputs.conf`:
```
[indexer_discovery:idx_discovery]
manager_uri = https://<CM>:8089
pass4SymmKey = idxforwarders
```

### Deployment Client
`deploymentclient.conf`:

```
[target-broker:deploymentServer]
targetUri = <DS-IP>:8089
```
### Data Inputs
UF1:
- Monitor Apache-style access logs → `network` index  
- Apply host extraction via transforms

  
---

## 📊 Monitoring Console

- Enable **Distributed Mode**
- Add search peers
- Configure Forwarder Monitoring
- Build the DMC asset table
