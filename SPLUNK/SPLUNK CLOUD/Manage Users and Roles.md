$${{\color{RoyalPurple}\Huge{\textsf{Manage Users and Roles}}}}\$$
---

${{\color{Silver}\large{\textsf{Description\ }}}}\$ <br>
---
In this exercise, you clone the sc_admin role to a new limited_admin role and modify its settings. Next, you modify the default index access permissions for the user and power roles. Then, you create two new users, assign them to roles, and test their respective level of access to Splunk Cloud features. Finally, you use the Cloud Monitoring Console to analyze user activity.

1. Direct your web browser to your splunk>cloud server 
2. Navigate to Settings > Roles.
3. In the settings column for the sc_admin role, click . > Clone.
   
<div align="center">
 <img width="600" height="300" alt="GetImage (8)" src="https://github.com/user-attachments/assets/dda2b428-4908-4301-8359-85066d6c1ff7" />
</div>


---

$${{\color{RoyalPurple}\Huge{\textsf{Assign explicit index access to the user and power roles.}}}}\$$

From splunk>cloud, navigate to: Settings > Roles

11. Click user.
12. Select the Indexes tab.
13. In the Included column, uncheck the checkbox for * (All non-internal indexes)
14. Check the Included checkboxes for the,bcg_online, itops, main and sales indexes as shown:

<div align="center">
<img width="600" height="300" alt="GetImage (9)" src="https://github.com/user-attachments/assets/f6a0ea4a-6ec4-468f-89f2-887a8488b97d" />
</div>



Verifying that the user: Cory can login

<div align="center">
<img width="206" height="77" alt="GetImage (10)" src="https://github.com/user-attachments/assets/e104fb2e-dcb3-4231-aec3-846865c48f5f" />
</div>

---

$${{\color{RoyalPurple}\Huge{\textsf{Add a user account for coryf and assign the limited-admin role to the account}}}}\$$

- Name: coryf
- Email address: coryf@example.com
- Set password: splunk3du
- Confirm password: splunk3du
- Assign to roles limited_admin (remove other roles in the Selected items list)
- Require password change… Uncheck this check box

---

$${{\color{RoyalPurple}\Huge{\textsf{Creating Power User }}}}\$$

- Name emaxwell
- Email address emaxwell@example.com
- Set password splunk3du
- Confirm password splunk3du
- Assign roles power
- Require password change… Disable this check box.
  
<div align="center">
  <img width="600" height="300" alt="GetImage (11)" src="https://github.com/user-attachments/assets/ee23c0db-2628-449a-9b5c-08b72191d0e0" />
</div>

---

$${{\color{RoyalPurple}\Huge{\textsf{Investigate user activities and usage in Splunk Cloud.}}}}\$$

<div align="center">
<img width="600" height="300" alt="GetImage (12)" src="https://github.com/user-attachments/assets/1f203ce2-ab6e-4816-986d-01f4488f754f" />


Clicking open in search for distinct users 


<img width="600" height="300" alt="GetImage (13)" src="https://github.com/user-attachments/assets/9737432d-bf6d-462e-bf38-3b440274797d" />
</div>

Clicking the search count and then selecting view events the additional search will be opened.
```
index=_internal (host=sh*.*splunk*.* OR host=si*.*splunk*.*) sourcetype=splunk_web_access method=GET status=200 uri_path="/*/app/*" user!=cmon_user file!="*.*"
| rex field=uri_path "\/app\/(?<App>[^\/]+).*"
| rename user as "User" file as "Page"
| stats count as Pageviews by User App Page
```
<div align="center">
<img width="600" height="300" alt="GetImage (14)" src="https://github.com/user-attachments/assets/ab7534b7-1381-48a2-8ead-51853b9b84fb" />
</div>
This search shows that apps that each users has accessed 

