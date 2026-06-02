
# Splunk Enterprise Security Lab
## Lab 1: Creating a SOC Manager Role, User, and Custom ES View

This lab introduces basic Splunk Enterprise Security (ES) administration tasks, including creating a custom SOC Manager role, assigning inherited capabilities, creating a SOC user account, filtering endpoint threats by urgency, and saving a custom view for analysts.

---

## 🎯 Objective
- Create a **SOC Manager** role that inherits permissions from the built‑in **ess_analyst** role.  
- Create a SOC user and assign the new role.  
- Explore endpoint threats with **High** or **Critical** urgency.  
- Save a custom filtered view for later use or sharing.  
- Understand how field selection impacts noise and analyst workflow.  
- Map severity values to urgency levels.

---

## 🧩 Task 1: Create the SOC Manager Role

### Role Requirements
The new role must:
- Inherit from **ess_analyst**  
- Automatically inherit all capabilities from:
  - `ess_analyst`
  - `ess_user`
  - `user`
 


### Steps
1. Navigate to:  
   **Settings → Access Controls → Roles**
2. Click **New Role**.
3. Name the role:  
   **soc_manager**
4. Under *Inherit From*, select:  
   **ess_analyst**
5. Save the role.

This ensures the SOC Manager has full ES analytical capabilities without manually assigning dozens of permissions.

<img width="1599" height="1257" alt="image" src="https://github.com/user-attachments/assets/be44c3ee-e688-4e84-a0bf-d0ab4017cde4" />
---

## 🧩 Task 2: Create the SOC User Account

1. Go to:  
   **Settings → Access Controls → Users**
2. Click **New User**.
3. Fill in:
   - Username: `soc_user`
   - Full Name: SOC Analyst / SOC Manager
   - Password: (your choice)
4. Assign the role:  
   **soc_manager**
5. Save the user.

The SOC user now has ES access and inherits all required capabilities.

<img width="1656" height="1313" alt="image" src="https://github.com/user-attachments/assets/fcb27fed-f646-4bb7-b578-0ef46c7cf584" />

---

## 🧩 Task 3: Investigate Endpoint Threats (High or Critical Urgency)

1. Log in as the SOC user.  
2. Navigate to:  
   **Enterprise Security → Incident Review**
3. Apply filters:
   - **Domain:** Endpoint  
   - **Urgency:** High, Critical  

This narrows the view to the most severe endpoint‑related threats, helping analysts prioritize investigations.

<img width="1625" height="1258" alt="image" src="https://github.com/user-attachments/assets/c258ed7b-489f-43ad-8529-2069d57c602c" />

---

## 🧩 Task 4: Create and Save a Custom View

After applying filters:

1. Click **Save As → View**.
2. Name the view:  
   **Endpoint High‑Urgency Threats**
3. Choose visibility:
   - **Private** (for personal use)  
   - **Shared** (for team use)

Saving views allows analysts to quickly return to filtered dashboards without reapplying filters.

---

## 🧩 Task 5: Adjust Fields to Reduce Noise

When reviewing the Incident Review table:

- Click **Edit Fields**.
- Remove fields that are rarely used by analysts.
- Keep only fields that support triage decisions (e.g., urgency, status, owner, rule name).

More fields ≠ better.  
Too many fields create noise and slow down investigations.  
A curated field set improves clarity and performance.

<img width="1648" height="801" alt="image" src="https://github.com/user-attachments/assets/b1480b39-aed4-4a19-88a1-8a85aa9a95b2" />

---

## 🧩 Task 6: Map Severity to Urgency

Splunk ES uses a mapping system to convert **severity** (from correlation searches) into **urgency** (used in Incident Review).

Example mapping:
- **Critical severity → Critical urgency**
- **High severity → High urgency**
- **Medium severity → Medium urgency**
- **Low severity → Low urgency**

This mapping ensures consistent prioritization across data sources and correlation searches.

You can view or modify this mapping in:  
**Configure → Content → Content Management → Adaptive Response Actions → Risk/Severity Mapping**

<img width="1632" height="1161" alt="image" src="https://github.com/user-attachments/assets/3aaf8fc8-fcf1-455e-98df-c02f99f8c422" />

---

## ✅ Lab Summary

By completing this lab, you have:

- Created a custom SOC Manager role with inherited ES capabilities.  
- Created a SOC user and assigned the new role.  
- Filtered endpoint threats by urgency to focus on high‑priority incidents.  
- Saved a reusable custom view for analysts.  
- Tuned field visibility to reduce noise.  
- Reviewed how severity maps to urgency in ES.

This forms the foundation for more advanced ES labs involving correlation searches, data model acceleration, and incident response workflows.
