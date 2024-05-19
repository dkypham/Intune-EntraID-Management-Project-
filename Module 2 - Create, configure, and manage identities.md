# Objectives
- Create a security group and assign a license
- Delete and restore a user
- Configure and manage device registration
- Create custom security attributes

Once users are authenticated, Microsoft Entra ID builds an access token to authorize the user and determine what resources they can access and what they can do with those resources.

Users are typically defined in three ways
- [Cloud identities] - exist only in Microsoft Entra ID
	- ex: administrator accounts and users that you manage yourself
	- their source is **Microsoft Entra ID** or **External Microsoft Entra directory**
- [Directory-synchronized identities] - exist in an on-premises Active Directory. Synchronization occurs Microsoft Entra Connect to bring users into Azure
	- their source is **Windows Server AD**
- [Guest users] - exist outside Azure. useful for external vendors or contractors who need access to your Azure resources
	- ex: from other cloud providers or Microsoft accounts such as Xbox LIVE.
	- their source is **Invited use**r.

Two types of groups
- [Security groups] - most common type of groups. used to manage member and computer access to shared resources for a group of users.
	- ex: create a security group for a specific security policy, given to all members at once
- [Microsoft 365 groups] - provide collaboration opportunities by giving members access to a shared mailbox, calendar, files, SharePoint site, and more.
Two types of membership
- [Assigned] - members are added and maintained manually
- [Dynamic] - members are added based on rules

[Microsoft Entra registered] - registered to Microsoft Entra ID without requiring organizational account to sign into the device
 - Primary audience - BYOD and mobile devices
 - Device ownership - User or Organization
 - Device management - Mobile Device Management (example: Microsoft Intune)
 - Key capabilities - SSO to cloud resources, Conditional access
[Microsoft Entra joined devices] - joined to Microsoft Entra ID requiring organizational account to sign into device
- Primary audience - cloud-only or hybrid organizations
- Device ownership - Organization
- Device management - Mobile Device Management (example: Microsoft Intune)
- Key capabilities - SSO to both cloud and on-premises resources, Conditional Access, Self-service Password Reset and Windows Hello PIN reset
[Hybrid Microsoft Entra joined] - joined to on-premises AD and Microsoft Entra ID requiring organizational account to sign in to device
- Primary audience - suitable for hybrid organizations with existing on-premises AD infrastructure
- Device ownership - Organization
- Device management - Group Policy, Configuration Manager standalone or co-management with Microsoft Intune
- Key capabilities - SSO to both cloud and on-premises resources, Conditional Access, Self-service Password Reset and Windows Hello PIN reset

# Device Writeback
For hybrid setups using Microsoft Entra Connect, [device writeback] allows on-premises AD to track devices registered in Microsoft Entra ID, enabling on-premises conditional access and supporting Windows Hello for Business in hybrid and federated environments

# Features of group-based licensing
- Licenses can be assigned to security groups in Microsoft Entra ID, which can be synced from on-premises or created directly in the cloud.
- Administrators can disable specific service plans when assigning licenses to groups.
- All Microsoft cloud services requiring user-level licensing, such as Microsoft 365, Enterprise Mobility + Security, and Dynamics 365, are supported.
- Group-based licensing is currently managed through the Azure portal.
- Microsoft Entra ID automatically updates licenses based on group membership changes, typically within minutes.
- Users can be members of multiple groups with different license policies and can also have directly assigned licenses, consuming each license only once.
- Administrators can access information about users for whom group licenses couldn't be fully processed and take corrective actions.

# License Assignment Errors
- Not enough licenses
	- occurs when there aren't enough available licenses for one of the products
	- Resolution: view how many licenses are available by going to Microsoft Entra - Identity - Billing, then Licenses, then All products
	- PowerShell error *CountViolation*
- Service plans that conflict
	- occurs when the group contains a service plan that conflicts with another service plan that's already assigned
	- Resolution: you need to disable one of the two plans by
		- disabling a license directly assigned to a user
		- modifying the entire group license assignment and disable the plans in the license
		- remove the license from the user if it's redundant
	- PowerShell error *MutuallyExclusiveViolation*
- Other products depend on this license
	- occurs when one of the products contains a service plan that must be enabled for another service plan to function
	- Resolution: make sure required plan is assigned to users, then remove group license as needed
	- PowerShell error *DependencyViolation*
- Usage location isn't allowed
	- occurs when service isn't available in a location because of local laws and regulations in specified User location property of user
	- Resolution: remove users from unsupported locations from the licensed group or modify user location
	- PowerShell error *ProhibitedUsageLocationViolation*
- Duplication proxy address
	- occurs when some users are incorrectly configured with the same proxy address value
	- Resolution: fix configuration
- Microsoft Entra Mail and ProxyAddress attribute change
	- occurs when updating license assignment on a user or group, Microsoft Entra Mail and ProxyAddresses attribute of some users are changed

# Migrating users with individual licenses to group licenses
1. Keep existing automation (for example PowerShell) managing license assignment and removal for users running as is.
2. Create a new licensing group (or decide which existing groups to use) and make sure that all required users are added as members.
3. Assign the required licenses to those groups; your goal should be to reflect the same licensing state as your existing automation
4. Verify that licenses have been applied to all users in those groups. This application can be done by checking the processing state on each group and by checking Audit Logs.
    - You can perform a random check of a few individual users by looking at their license details. You will see that they have the same licenses assigned “directly” and “inherited” from groups.
    - When the same product license is assigned to the user both directly and through a group, only one license is consumed by the user. Hence no additional licenses are required to perform migration.
5. Verify that no license assignments failed by checking each group for users in error state.

# [Custom security attribute] - business-specific attributes (key-value pairs that you can define and assign to Microsoft Entra objects
- Use case
	- extend user profiles
		- ex: employee hire date, hourly salary, etc
		- ensure only administrators can see certain attributes
		- categorize hundreds or thousands of applications to easily create a filterable inventory for auditing

# Automatic user creation
- Components of system SCIM (System for Cross-Domain Identity Management)
	- [HCM system] - Applications and technologies that enable Human Capital Management process and practices that support and automate HR processes throughout the employee lifecycle.
	- [Microsoft Entra Provisioning Service] - Uses the SCIM 2.0 protocol for automatic provisioning. Connects to the SCIM endpoint for the application, and uses the SCIM user object schema and REST APIs to automate provisioning and de-provisioning of users and groups.
	- [Microsoft Entra ID] - User repository used to manage the lifecycle of identities and their entitlements.
	- [Target system] - Application or system that has SCIM endpoint and works with the Microsoft Entra provisioning to enable automatic provisioning of users and groups.