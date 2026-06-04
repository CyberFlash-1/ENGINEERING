$${{\color{RoyalPurple}\Huge{\textsf{Splunk Cloud LDAP Configuration and User Mapping Lab}}}}\$$
---
${{\color{Silver}\large{\textsf{Objective}}}}\$ <br>
The objective of this lab was to configure LDAP authentication in Splunk Cloud, map Active Directory groups to Splunk roles, verify successful synchronization of users from Active Directory, and test LDAP-based user authentication.


---

## LDAP Configuration

The LDAP configuration was completed within the Splunk Cloud environment to allow authentication against the organization's Active Directory. The configuration included:

<div align="center">
<img width="600" height="300" alt="GetImage (15)" src="https://github.com/user-attachments/assets/48f961ed-6665-49b5-9156-e497b4ec8b81" />
</div>

- Defining the LDAP server connection details.
- Establishing the bind account.

<div align="center">
<img width="600" height="300" alt="GetImage (16)" src="https://github.com/user-attachments/assets/53af5dec-4f5c-495f-b558-ebbe60e4338a" />
</div>
- Configuring user and group search bases.
- Enabling LDAP authentication.

<div align="center">
<img width="800" height="300" alt="GetImage (17)" src="https://github.com/user-attachments/assets/9da31377-f6f8-4470-b2cd-9ea154dbee28" />
</div>

After the LDAP settings were applied, the connection was validated to ensure that Splunk could successfully communicate with Active Directory and retrieve user and group information.

---

## Group Mapping Configuration

The next step involved configuring role mappings between Active Directory security groups and Splunk roles.

The initial mapping was established as follows:

<div align="center">
<img width="800" height="300" alt="GetImage (18)" src="https://github.com/user-attachments/assets/66876761-8e5a-4795-b3a7-399622ff5feb" />
</div>

- Active Directory groups were imported into Splunk.
- Each group was assigned the appropriate Splunk role based on organizational access requirements.
- Role mappings ensured that users would automatically inherit the correct permissions upon successful LDAP authentication.
  
<div align="center">
<img width="800" height="300" alt="GetImage (19)" src="https://github.com/user-attachments/assets/31a8932e-9ab6-49b3-adf1-8a1639459408" />
</div>

This configuration simplifies access management by allowing administrators to manage permissions through Active Directory group membership rather than maintaining separate local Splunk accounts.

---

## Verification of LDAP Configuration

Once the LDAP integration and role mappings were configured, verification testing was performed to confirm that Active Directory users were being successfully imported into Splunk.

<div align="center">
<img width="800" height="400" alt="GetImage (20)" src="https://github.com/user-attachments/assets/ca193547-90f0-47d5-94e4-52a2561a91a6" />
</div>


### User Import Verification

The LDAP synchronization process successfully imported user accounts from Active Directory.

**Result:**

- Total users imported from Active Directory: **9 users**

This confirms that the LDAP search configuration was correctly identifying and retrieving users from the specified Active Directory organizational units.

---

## Role Mapping Verification

The imported users were reviewed to confirm that role mappings were applied correctly.

### Users Mapped to the User Role

The following LDAP users were successfully assigned to the **user** role:

| Full Name     | Username |
|---------------|----------|
| Bao Lu        | blu      |
| Dwight Hale   | dhale    |

This verified that the configured group-to-role mappings were functioning as expected and that users received the appropriate Splunk permissions based on their Active Directory group membership.

---

## Authentication Testing

To validate end-user access, an LDAP-authenticated login test was performed using the Active Directory account belonging to Naomi Sharpe.

**Login Test Steps:**

1. Navigate to the Splunk Cloud login page.
2. Enter Naomi Sharpe's Active Directory credentials.
3. Submit the authentication request.
4. Verify successful login to the Splunk environment.

<div align="center">
<img width="800" height="400" alt="GetImage (21)" src="https://github.com/user-attachments/assets/813e1c07-ad9b-4c66-a8ba-ae8c5aca069c" />

</div>


**Results:**

- After authentication, the Splunk interface loaded successfully.
- The user account information displayed in the upper-right corner confirmed that the session was authenticated using Naomi Sharpe's LDAP account.

This successful login demonstrated that:

- LDAP authentication was functioning correctly.
- Active Directory credentials were accepted by Splunk Cloud.
- User synchronization and role mapping were operating as expected.
- End users could successfully access Splunk using their corporate credentials.

---
<div align="center">
<img width="800" height="400" alt="GetImage (22)" src="https://github.com/user-attachments/assets/9f5ca084-a3da-4009-aaf3-d5ebf3032352" />
</div>

## Conclusion

The LDAP integration with Splunk Cloud was successfully configured and validated. Active Directory users were imported correctly, group-to-role mappings were applied as intended, and authentication testing confirmed that users could log in using their Active Directory accounts.  

- Total users synchronized from Active Directory: **9**  
- Role assignments verified for selected users: Bao Lu and Dwight Hale  
- Successful login test confirmed end-to-end functionality.
