# Intro into Entra Global Secure Access

In this write-up I will dive into one of the two components that make up the Entra Global Secure Access suite: Entra Private Access.

As a quick intro, what is Entra Global Secure Access (GSA) and how do its two main components stack up ? 

[Intro into GSA](https://github.com/verboompj/EntraGSA/blob/main/README.md#intro-into-gsa)

[Setup Entra Private Access](https://github.com/verboompj/EntraGSA/blob/main/README.md#setup-entra-private-access)




## Intro into GSA

Entra GSA is Microsoft's SSE (Security Service Edge) Solution and consists of 2 main services:
- Entra Internet Access 
- Entra Private Access.

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/global-secure-access-diagram.png)



### Entra Internet Access - 
Microsoft Entra Internet Access secures access to Microsoft 365, SaaS, and public internet apps while protecting users, devices, and data against internet threats.
It provides an identity-centric Secure Web Gateway (SWG) solution for Software as a Service (SaaS) applications and other Internet traffic. 

#### Key features ( amongst others) 
- Prevent stolen tokens from being replayed with the compliant network check in Conditional Access.
- Apply universal tenant restrictions to prevent data exfiltration to other tenants or personal accounts including anonymous access.
- Enriched logs with network and device signals currently supported for SharePoint Online traffic.
- Protect user access to the public internet while leveraging Microsoft's cloud-delivered, identity-aware SWG solution.
- Enable web content filtering to regulate access to websites based on their content categories and domain names.
- Apply universal Conditional Access policies for all internet destinations, even if not federated with Microsoft Entra ID, through integration with Conditional 
  Access session controls.

i.e. an Identity (& Device) based upstream Secure Web Gateway as part of the Entra platform, leveraging CASB and Conditional Access components of that very suite, tunneling specified traffic over a SWG to its destination using a secured and conditioned path. 

### Entra Private Access - 
Microsoft Entra Private Access provides your users - whether in an office or working remotely - secured access to your private, corporate resources. Microsoft Entra Private Access builds on the capabilities of Microsoft Entra application proxy and extends access to any private resource, port, and protocol.

#### Key features
- Quick Access: Zero Trust based access to a range of IP addresses and/or FQDNs without requiring a legacy VPN.
- Per-app access for TCP apps (UDP support in development).
- Modernize legacy app authentication with deep Conditional Access integration.
- Provide a seamless end-user experience by acquiring network traffic from the desktop client and deploying side-by-side with your existing third-party SSE solutions.

i.e. Azure AD ( Entra) App Proxy on Steroids. This is a service of particular interest to me personally, as I love to phase out any form of (Client)VPN service wherever I see them. This is that very VPN killer service that promises to accomplish just that. 
It also leverages Entra's other services such as conditional access framework, MFA, RBAC, etc. Lets explore.

###
###

## Setup Entra Private Access

The Entra Private Access service consists of 3 main components:
1. A Client - installers available for Windows, Android, IOS and macOS - installed on the client device.
  
2. A Connector - This is a (group of) server(s) that has a line of sight to the service one wants to expose - a web, rds, ssh, whatever service you want your client to be able to connect to. On these servers one installs the Connnector Service to reverse-connect into the Entra platform. 

3. An Entra Application registration, representing the service you'd like to expose, the conditional access policy and the user asignment. This is the " traditional" Entra Enterprise Application as we know it.

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/private-access-diagram-quick-access2.png)








