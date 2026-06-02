***This lab component is an extension of Splunk Enterprise Deployment Lab***

## Install Active Directory Domain Services
What is a Domain Controller?
A Domain Controller (DC) is a server that runs Active Directory. It is the brain of the entire identity system. When a user logs in anywhere on the domain, their credentials are checked against the Domain Controller. When you create a user account, you create it on the Domain Controller. There is usually more than one in an enterprise for redundancy, but we are building one here. Everything that joins your network will trust this server to make authentication decisions.


<img width="400" height="300" alt="image" src="https://github.com/user-attachments/assets/dbc8d892-a6ac-4f51-a87b-d01cd912b14e" />

In Server Manager, click Manage → Add Roles and Features. Click Next through the wizard until you reach Server Roles. Check Active Directory Domain Services. When prompted, click Add Features to include the management tools. Click Next through the remaining pages and click Install. Wait for installation to complete — takes 2–3 minutes. When complete, click Close — do not restart yet

# Alternatively, run this in PowerShell to install the role:
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Run this in PowerShell immediately after installing AD DS
```
Install-WindowsFeature -Name GPMC
```
# Wait for it to complete, then close and reopen Server Manager

## Promote the Server to a Domain Controller
- In Server Manager, click the yellow warning flag at the top right
- Click Promote this server to a domain controller
- Select Add a new forest
- Set Root domain name to: lab.local
- Click Next — set a DSRM password (write it down — needed for disaster recovery only) - Administrator123!
- 
- Click through DNS Options and NetBIOS pages — accept the defaults
- Click Install — the server will automatically restart when complete

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/fc9a7662-c7ce-46fe-acaa-903ad17eccb0" />

## Build the Organizational Structure
OUs to organise users, computers, and groups by department, location, or function — whatever makes sense for your organization. The real power of an OU is that you can link a Group Policy to it. That means every user or computer inside that OU automatically gets the policy applied.
```
# PowerShell — create all 5 OUs at once
New-ADOrganizationalUnit -Name "Splunk"   -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Group"    -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Users"    -Path "DC=lab,DC=local"
```

## Create Security Groups 
A Security Group is a container that holds user accounts. Instead of granting access to a resource (like a file share or an application) to individual users one by one, you grant access to a group
```
# PowerShell — create all 4 security groups
New-ADGroup -Name "SplunkPowerUsers"     -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=lab,DC=local"
New-ADGroup -Name "SplunkAdmins"      -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=lab,DC=local"
New-ADGroup -Name "SplunkUsers"      -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=lab,DC=local"
```

## Configure Group Policy Management
Group Policy is how you enforce settings across every machine and user in the domain without logging into each one individually. Open Group Policy Management from the Tools menu in Server Manager.

Expand Forest: lab.local → Domains → lab.local in Group Policy Management
Right-click the IT OU → Create a GPO in this domain and link it here
Name it: IT Security Policy
Right-click the new GPO → Edit
Configure the settings in the table below

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/5aff4339-0cde-4cdc-afcb-bb0c37e4249e" />

### Defining the minimum password length
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/a96c1146-706b-4592-94d4-7051bf9b0c09" />
<br>

```
net accounts /lockoutthreshold:5 /domain
```
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/88e684cf-cb18-4119-b8d8-b3d237d33fd2" />

### GPO is successfully being enforce 
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/9c59ae74-7e45-404b-8ef4-8da6d3462288" />

### DistinguishedNAME 



<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/872e4e1c-b072-4d0e-b6d6-cfc70b1c1bd4" />

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/bae7c405-3d64-48c4-a832-5906eb48c2c1" />

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/f9db19ef-307c-437e-b0ee-089ebeaaa9e6" />
