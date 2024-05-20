# Objective - securely invite external users and manage their accounts and configure identity providers
- configure external collaboration settings
- invite external users - individually and in bulk
- create dynamic group

[Microsoft Entra B2B collaboration] - collaboration tool with guest users. Securely share company's applications and services with external users while maintaining control over your own corporate data 
# Invitation redemption flow
![[business-to-business-invitation-redemption-3caadd6b.png]]
1. Perform user-based discovery to determine if the user already exists in a managed Microsoft Entra tenant
2. If admin enabled SAML/WS-Fed IdP federation
	- check user's domain suffix for a match to redirect to a pre-configured identity provider
3. If admin enabled Google federation
	- check if user has an existing MSA, use that to sign in
4. Once user's home directory is identified, user is sent to the corresponding identity provider to sign in
	- If no home directory is found AND email one-time passcode is enabled for guests, email a passcode
	- If no home directory is found AND email one-time passcode is disabled for guests, prompt user to create a consumer MSA
- After authenticating to the right identity provider, redirect to Microsoft Entra ID to complete the consent experience3

[Microsoft Entra entitlement management] - configure policies that manage access for external users
# B2B external collaboration settings
- turn off invitations (no external users can be invited)
- only admins and users in the Guest Inviter role can invite
- admins, the guest inviter role, and members can invite
- all users, including guests, can invite
- by default, all users, including guests, can invite guest users
# Invite external users
- guest users can be invited to a directory, group, or application
- once invited, the user's account is added to Microsoft Entra ID with user type of Guest
- invitation must be redeemed to access resources and the invitation does not expire
- application owners can manage guest access and send direct app links through self-service app management for gallery and SAML-based apps with some initial setup
	- enable self-service group management for your tenant
	- create a group to assign to the app and make the user an owner
	- configure the app for self-service and assign the group to the app
- to bulk invite use a .csv
# CSV template structure
- rows
	- version number
	- column headings
- required columns are listed first

| version:v1.0                                    |                                              |
| ----------------------------------------------- | -------------------------------------------- |
| Email address to invite [inviteeEmail] Required | Redirection url [inviteRedirectURL] Required |
| example@example.com                             | https://myapps.azure.com                     |
# Manage external user accounts in Microsoft Entra ID
- Key properties of the Microsoft Entra B2B collaboration user
	- UserType
		- Member - employee of host org and user in the org's payroll
		- Guest - isn't considered internal to company
	- Identities: primary identity provider
		- External Microsoft Entra tenant - user is homed in an external organization and authenticates by using a Microsoft Entra account that belongs to the other organization
		- Microsoft account - user is homed in a Microsoft account and authenticates by using a Microsoft account
		- {host's domain}
		- google.com
		- facebook.com
		- mail - user has signed up by using OTP
		- {issuer URI} - user is homed in an external organization that doesn't use Microsoft Entra ID as their identity provider, but instead uses a SAML/WS-Fed-based identity provider
[dynamic groups] - groups populated based on user attributes (such as userType, department, country/region)
# Implement and manage Microsoft Entra Verified ID
[Microsoft Entra Verified ID] - decentralized identity solution that allows organizations to issue and verify customized verifiable credentials, providing secure and seamless identity management through a free REST API.
- requirements
	- An Azure tenant with a subscription
	- A Microsoft Entra ID premium license
	- Logged in as the global administrator
	- A configured Azure Key Vault instance
# Configure Identity providers
- [Direct federation] - used to be called [SAML/WS-Fed identity provider federation]. Guest users will sign into your Microsoft Entra tenant using their own organizational account. When accessing shared resources, they are prompted for sign-in and redirected to their IdP.
	- when setup, any new guest users invited will be authenticated using that SAML/WS-Fed IdP
		- guests will use same authentication method they used before organization's federation was set up
		- if federation is deleted with an organization's SAML/WS-Fed IdP, any guest using it will be unable to sign in
- SAML 2.0 Configuration
	- Required attributes the IdP must send back in the SAML

| Attribute                | Value                                                                                        |
| ------------------------ | -------------------------------------------------------------------------------------------- |
| AssertionConsumerService | `https://login.microsoftonline.com/login.srf`                                                |
| Audience                 | `urn:federation:MicrosoftOnline`                                                             |
| Issuer                   | The issuer URI of the partner IdP, for example `https://www.example.com/exk10l6w90DHM0yi...` |
	- Required details in the SAML token provided by the IdP

| **Attribute** | **Value**                                                             |
| ------------- | --------------------------------------------------------------------- |
| NameID Format | `urn:oasis:names:tc:SAML:2.0:nameid-format:persistent`                |
| emailaddress  | `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` |
- WS-Federation configuration
	- Required attributes in the WS-Fed message from the IdP

| **Attribute**            | **Value**                                                                                    |
| ------------------------ | -------------------------------------------------------------------------------------------- |
| PassiveRequestorEndpoint | `https://login.microsoftonline.com/login.srf`                                                |
| Audience                 | `urn:federation:MicrosoftOnline`                                                             |
| Issuer                   | The issuer URI of the partner IdP, for example `https://www.example.com/exk10l6w90DHM0yi...` |

	- Required claims for the WS-Fed token issued by the IdP

| **Attribute** | **Value**                                                             |
| ------------- | --------------------------------------------------------------------- |
| ImmutableID   | `https://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID` |
| emailaddress  | `https://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` |
# Implement cross-tenant access controls
- Inbound and outbound settings

| **Cross-tenant access setting name** | **Operations managed**                                                                                                                                                                                                                                                                                                                 |
| ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Outbound access settings             | Control whether users can access resources in an external organization. You can apply settings to everyone, or specify individual users, groups, and applications.                                                                                                                                                                     |
| Inbound access settings              | Control whether users from external Microsoft Entra organizations can access resources in your organization. You can apply these settings to everyone, or specify individual users, groups, and applications.                                                                                                                          |
| Trust settings (inbound)             | Determine whether your Conditional Access policies will trust the multifactor authentication (MFA). You can also require compliant device, and hybrid Microsoft Entra joined device. And finally, allow or restrict user from an external organization if their users have already satisfied these requirements in their home tenants. |
| B2b direct connect                   | Set up a mutual trust relationship with another Microsoft Entra organization for seamless collaboration. This feature currently works with Microsoft Teams shared channels.                                                                                                                                                            |


- [Microsoft cloud specific configuration] - for government contracts connect to Microsoft Azure Government or Microsoft Azure China
- [B2B Direct Connect] - both the resource organization and the external organization need to mutually enable. Once trust is established, the B2B direct connect user has SSO access to resources using credentials from their Microsoft Entra organization