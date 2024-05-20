# Objectives
- plan, design, and implement Microsoft Entra Connect
- implement manage password hash synchronization (PHS)
- implement manage pass-through authentication (PTA)
- implement and manage federation

# Implementing Microsoft Entra Connect
- bridges an organizations on-premises Active Directory with your cloud-based Microsoft Entra ID
- synchronize identities from on-premises into Azure
- consistent identity across both platforms
- Capabilities
	- [Synchronization] - responsible for creating users, groups, and other objects. Then, making sure identity information for your on-premises users and groups is matching the cloud. also includes password hashes
	- [Password hash synchronization] - sign-in method that synchronizes a hash of a user's on-premises AD password with Microsoft Entra ID
	- [Pass-through authentication] - sign-in method that allows users to use the same password on-premises and in the cloud, but doesn't require the extra infrastructure of a federated environment
	- [Federation integration] - optional part of Microsoft Entra Connect and can be used to configure a hybrid environment using an on-premises AD FS infrastructure. Also provides AD FS management capabilities such as certificate renewal and more AD FS server deployments
	- [Health monitoring] 
- single identity to access on-premises applications and cloud services such as Microsoft 365
- Identity control plane
	- consider time, existing infrastructure, complexity, and cost of implementing your choice
# Cloud authentication
- [Microsoft Entra password hash synchronization] or [PHS] - simplest way to enable authentication for on-premises directory objects in Microsoft Entra. Users can use the same username and password that they use on-premises without having to deploy more infrastructure.
	- [Effort] - sync process and run every two minutes
	- [User experience] - seamless SSO with password hash synchronization
	- [Advanced scenarios] - use insights from identities with Microsoft Entra Identity Protection reports. 
	- [Business continuity] - to make sure password hash sync doesn't go down for extended periods, deploy a second Microsoft Entra Connect server in staging mode in a standby configuration
	- [Considerations] - password hash sync doesn't immediately enforce changes in on-premises account states. Admins should consider running a new sync cycle after bulk updates
		- ex: disabling accounts
- [Microsoft Entra pass-through authentication] or [PTA]
	- [Effort] - need one or more lightweight agents installed on existing servers that have access to on-premises AD Domain Services, including on-premises AD domain controllers. Need outbound access to the Internet. 
	- [User experience] - seamless SSO with pass-through authentication
	- [Advanced scenarios] - enforces the on-premises account policy at the time of sign-in. Access can be denied if sign-in attempt falls outside the hours user is allowed to sign in
	- [Business continuity] - recommend deploying two extra pass-through authentication agents to ensure high availability of authentication requests. One agent can fail while another is down for maintenance
	- [Considerations] - hash sync can be used as a backup when agents can't validate a user's credentials due to a significant on-premises failure.
- [Federated authentication] - hand off authentication process to a separate trusted authentication system
	- such as on-premises AD Federation Services
	- [Effort] - rely on external trusted system
	- [User experience] - depends on implementation
	- [Advanced scenarios] - required when customers have an authentication requirement that Microsoft Entra ID doesn't support natively
		- smartcards or certificates
		- on-premises MFA servers or third-party multifactor providers requiring a federated identity provider
		- authentication by using third-party authentication solutions
		- sign in that requires a sAMAccountName instead of a User Principal Name (UPN)
			- ex: DOMAIN\username instead of user@domain.com
	- [Business continuity] - typically require load-balanced array of servers, aka a farm
	- [Considerations] - typically require more significant investment in on-premises infrastructure
# Topologies for Microsoft Entra Connect
| **Topology**                                                     | **Description**                                                                                                                                                                                                           |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Single forest, single Entra tenant**                           | One on-premises forest with one or more domains and one Entra tenant. Uses password hash synchronization for authentication. This is the simplest and most common setup.                                                  |
| **Multiple forests, single Entra tenant**                        | Multiple on-premises AD forests connected to one Entra tenant. Common in organizations with complex environments, like mergers. All forests must be accessible by a single Entra Connect sync server.                     |
| **Multiple forests, single sync server, users in one directory** | Each forest operates independently with no users present in other forests. Each has its own Exchange setup, and no global address list (GAL) synchronization between forests. They appear unified in Entra ID.            |
| **Multiple forests: full mesh with optional GALSync**            | Users and resources can be in any forest with two-way trusts. Optionally, GALSync synchronizes the address list across forests. Microsoft Entra Connect can't handle on-premises GALSync.                                 |
| **Multiple forests: account-resource forest**                    | A resource forest trusts all account forests, typically containing shared services like Exchange and Teams. Users have a disabled account in this forest linked to their account forest for mailboxes and other services. |
| **Staging server**                                               | An extra Entra Connect server in staging mode reads but doesn't write data. Keeps an updated copy of identity data using the normal sync cycle, useful for backup or failover.                                            |
| **Multiple Entra tenants**                                       | Each Entra tenant requires a separate Entra Connect sync server. Tenants are isolated, so users in one tenant can't see users in another. This setup is for maintaining strict user separation.                           |
| **Each object only once in a tenant**                            | One Entra Connect sync server per tenant, each configured to manage a distinct set of objects. For instance, each server can handle specific domains or organizational units without overlap.                             |
# Microsoft Entra Connect component factors
![[azure-active-directory-connect-internal-0444044d.png]]
- [Connector Space] (CS): This is the staging area for objects from each connected directory (CD) before they are processed by the provisioning engine. Microsoft Entra ID and each connected forest have their own CS.
- [Metaverse] (MV): This is where objects to be synchronized are created based on sync rules. Objects must exist in the MV to be synchronized with other connected directories. There is only one MV.
- [Sync Rules]: These rules determine which objects are created or connected in the MV and which attribute values are copied or transformed between directories.
- [Run Profiles]: These profiles bundle the process steps for copying objects and their attribute values according to sync rules between the staging areas and connected directories.
# Microsoft Entra cloud sync
- Support for synchronizing to a Microsoft Entra tenant from a multi-forest disconnected Active Directory forest environment: Useful for scenarios like mergers and acquisitions, where the acquired company's AD forests are isolated from the parent company's AD forests, or in companies with multiple AD forests.
- Simplified installation with light-weight provisioning agents: These agents act as a bridge from AD to Microsoft Entra ID, with all synchronization configuration managed in the cloud.
- Multiple provisioning agents: Enable high availability deployments, which is critical for organizations relying on password hash synchronization from AD to Microsoft Entra ID.
- Support for large groups with up to fifty-thousand members: When synchronizing large groups, it is recommended to use only the OU scoping filter.
# Requirement to deploy federation with AD FS and Microsoft Entra Connect
- Local administrator credentials on your federation servers.
- Local administrator credentials on any workgroup servers (not domain-joined) that you intend to deploy the Web Application Proxy role on.
- The machine that you run the wizard on to be able to connect to any other machines that you want to install AD FS or Web Application Proxy on by using Windows Remote Management.
- Steps
	- specify the AD FS servers - where you want to install AD FS. 
	- specify the Web Application Proxy servers - deployed in your perimeter network, facing the extranet. supports authentication requests from the extranet
	- specify the service account for the AD FS service - requires a domain service account to authentication users and to look up user information in Active Directory. Supports two types of service accounts
		- Group managed service account
		- Domain user account
	- select the Microsoft Entra domain that you want to federate - set up the federation relationship between AD FS and Microsoft Entra ID. configure AD FS to provide security tokens to Microsoft Entra ID
# Synchronization errors
- InvalidSoftMatch
	- Microsoft Entra Connect uses **Hard Match** to match objects based on the sourceAnchor attribute to the immutableId in Microsoft Entra ID. If no match is found, it uses **Soft Match** based on ProxyAddresses and UserPrincipalName. An **InvalidSoftMatch error** occurs when a Soft Match finds an object, but its immutableId differs from the incoming object's sourceAnchor, indicating the object was synced with another on-premises object.
- ObjectTypeMismatch
	- when two objects of different "object type" such as User, Group, Contact have the same values for the attributes
- AttributeValueMustBeUnique
	- Microsoft Entra schema does not allow two or more objects to have the same value of the following attributes
		- ProxyAddresses
		- UserPrincipalName
- IdentityDataValidationFailed
	- attribute value has invalid/unsupported characters
- FederatedDomainChangeError
	- when the suffix of a user's UserPrincipalName is changed from one federated domain to another federated domain
- LargeObject
	- when an attribute exceeds the allowed size limit, lengt limit, or count limit
	- typically occurs for the following attributes: userCertificate, userSMIMECertificate, thumbnailPhoto, proxyAddresses
# Manage Microsoft Entra Health
- manage email notifications
- delete a server or service instance
- delete a server from the Health service
- delete a service instance from Health service
- Azure role-based access control

|**Role**|**Permissions**|
|---|---|
|Owner|Owners can _manage access_ (for example, assign a role to a user or group), _view all information_ (for example, view alerts) from the portal, and _change settings_ (for example, email notifications) within Microsoft Entra Connect Health. By default, Microsoft Entra global administrators are assigned this role, and this cannot be changed.|
|Contributor|Contributors can _view all information_ (for example, view alerts) from the portal, and _change settings_ (for example, email notifications) within Microsoft Entra Connect Health.|
|Reader|Readers can _view all information_ (for example, view alerts) from the portal within Microsoft Entra Connect Health.|
