# Objective - configure and manage a Microsoft Entra tenant
- create custom role "Test Role"
- sign in as a Global administrator and create a new user "Chris Green"
	- password "Luro572798"
	- set their role to "Application Administrator"
	- remove their role of "Application Administrator"
- add custom domain names and subdomains to your Microsoft Entra organization
- configure external user options
- configure tenant properties such as display name, country, ID, technical contact, global privacy contact, privacy statement

- Configure and manage Microsoft Entra roles.
- Configure and manage custom domains.
- Evaluate permissions based on role assignments and settings.
- Configure delegation by using administrative units.
- Configure tenant-wide settings such as display name, country, ID, technical contact, global privacy contact, privacy statement
# Notes
[Microsoft Entra ID] - Microsoft's cloud-based identity and access management service.
- allows employees to access external resources such as Microsoft 365 or SaaS applications and internal resources such as corporate network and intranet
- IT admins control access to apps and app resources
	- implement MFA, automate user provisioning
# Microsoft Entra roles
[Global Administrator]
- Manage access to all administrative features in Microsoft Entra ID, and services that federate to Microsoft Entra ID
- Assign administrator roles to others
- Reset the password for any user and all other administrators
[User Administrator]
- Create and manage all aspects of users and groups
- Manage support tickets
- Monitor service health
- Change passwords for users, Helpdesk administrators, and other User Administrators
[Billing Administrator]
- Make purchases
- Manage subscriptions
- Manage support tickets
- Monitor service health
# Difference between Azure and Microsoft Entra
![[azure-office-roles-567e92bc.png]]
By default, Azure roles and Microsoft Entra roles are separate, but a Global Administrator can gain the User Access Administrator role in Azure by enabling the Access management for Azure resources switch. While Global Administrators can manage both Microsoft Entra ID and Microsoft 365 services, they don't have Azure access unless they elevate their permissions.
# Ways to assign roles
- Assign a role to a user or group
	- Microsoft Entra ID - Roles and administration - Select a role - + Add Assignment
- Assign a user or group to a role
	- Microsoft Entra ID - Open Users (or Groups) - Select an User (or group) - Assigned roles - + Add Assignment
- Assign a role to a broad-scope, like a Subscription, Resource Group, or Management Group
    - Done via the **Access control (IAM)** within each settings screen
- Assign a role using PowerShell or Microsoft Graph API
- Assign a role using Privileged Identity Management (PIM)

[Administrative units] - An administrative unit in Microsoft Entra ID is a container that allows you to limit administrative roles to specific users or groups, ensuring administrators can only manage designated subsets of users, thereby adhering to the principle of least privilege. 
- roles that can manage administrative unit: 
	- Authentication administrator
	- Groups administrator
	- Helpdesk administrator
	- License administrator
	- Password administrator
	- User administrator
- creation stages
	1. Initial adoption - create administrative units based on initial criteria
	2. Pruning - administrative units that are no longer required will be deleted
	3. Stabilization - once organizational structure is defined, the number of administrative units won't change significantly
# Delegate administration in Microsoft Entra ID
- restrict who can create applications and manage the applications they create
	- by default all users can register application registrations and manage all aspects of applications they create
- assign one or more owners to an application
- assign a built-in administrative role that grants access to manage configuration in Microsoft Entra ID for all applications
- Make a new role with certain powers. Then give it to someone as a limited owner. Or give it to a bunch of apps as a limited admin.
# Recommended Steps for Delegation
1. Define the roles you need
2. Delegate app administration
3. Grant the ability to register applications
4. Delegate app ownership
5. Develop a security plan
6. Establish emergency accounts
7. Secure your administrator roles
8. Make privileged elevation temporary
# Most privileged roles
- [Application Administrator] - manage all applications in the directory, including registrations, SSO settings, user and group assignments and licensing, Application proxy settings, and consent
	- doesn't grant the ability to manage Conditional Access
- [Cloud Application Administrator] - all abilities of the Application Administrator except Application Proxy settings
# Delegate app ownership
- [Enterprise Application Owner] - manage the 'enterprise applications that the user owns', including SSO settings, user and group assignments, and adding more owners.
	- doesn't grant the ability to manage Application Proxy settings or Conditional Access
- [Application Registration Owner] - manage application registrations for apps that the user owns, including the application manifest and adding other owners

Configure tenant-wide setting
- [tenant-wide setting] - configuration options that apply to all resources within your tenant
- tenant properties - where you give the name of your directory and set values like the primary contact
- user settings - where you define what global rights your users have, like registering applications
- external collaboration settings - where you define what task an external guest user can perform like inviting more guest users