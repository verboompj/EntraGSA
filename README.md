# Intro into Entra Global Secure Access

In this write-up I will dive into one of the two components that make up the Entra Global Secure Access suite: Entra Private Access.

As a quick intro, what is Entra Global Secure Access (GSA) and how do its two main components stack up ? 

[Intro into GSA](https://github.com/verboompj/EntraGSA/blob/main/README.md#intro-into-gsa)

[Setup Entra Private Access](https://github.com/verboompj/EntraGSA/blob/main/README.md#setup-entra-private-access)

- [1. Connector Service](https://github.com/verboompj/EntraGSA/blob/main/README.md#2-connector-service)
- [2. Entra App registration](https://github.com/verboompj/EntraGSA/blob/main/README.md#2-entra-app-registration)
- [3. Global Secure Access Client](https://github.com/verboompj/EntraGSA?tab=readme-ov-file#3-global-secure-access-client)

[Testing](https://github.com/verboompj/EntraGSA/blob/main/README.md#testing-in-practice)


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
  
1. A Connector - This is a (group of) server(s) that has a line of sight to the service one wants to expose - a web, rds, ssh, whatever service you want your client to be able to connect to. On these servers one installs the Connnector Service to reverse-connect into the Entra platform. 

2. An Entra Application registration, representing the service you'd like to expose, the conditional access policy and the user asignment. This is the " traditional" Entra Enterprise Application as we know it + Network Access properties.

3. A Client - Global Secure Access Client - installers available for Windows, Android, IOS and macOS - installed on the client device.

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/private-access-diagram-quick-access3.png)


#### 1. Connector Service

The connector service, in my example, is a Windows Server deployed in Azure. It is deployed in the same vnet , however on a different subnet, than the service I'd like to publish/expose using Entra Private access. In my case I'd like to expose a set of Linux Virtual machines over SSH. 

Non of my deployed servers has a public IP address. In this case my Windows server does have outbound Internet connectivity using a NAT gateway. No port forwards have been configured, and only outbound connectivity is allowed.

I want users to be able to connect to these Linux servers over SSH without setting up any form of network connectivity using traditional service such as VPN. Instead, I'd like my users to connect from their local devices to the private IP addresses of these Linux VM's, directly over the internet. In their case any wifi / 5G connection available.

In order to achieve this, my Windows Server is confirmed to have line of sight with the Linux servers, has outbound Internet connectivity and is equiped with the (Entra) Connector Service. As soon as the service is installed, the server pops up in the Entra portal as an available connector: 

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/connectors2.png)

Here one can group multiple connectors into a group for redundancy purposes. For production a minimum of 2 servers is recommended. The servers need to share the common basics such as connectivity to the "to be exposed" service(s). 

#### 2. Entra App Registration

To define the exposed service one can create an Enterprise Application, name it, select the Connector Group to be used ( step 2) and under Application Segment, define an Application Segment, where one defines the (private) FQDN or rfc1918 IP-address and the Portnumber of the service. In this case, for SSH, its port 22. 
Other options are a complete CIDR range or a custom IP-to-IP range and Port combination.  

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/EntraAppEnt2.png)

Next, we have to setup a Traffic Forwarding Profile. This enables our configuration of the Entra Private Access Enterprise App to flow to the clients in the field. 

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/forwarding.png)

#### 3. Global Secure Access Client

Installing the client is straight forward, there are a few catches, such as the client device having to be either Entra or Hybrid joined ( not registered) The requirements are listed here: https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-install-windows-client#prerequisites

Once setup, you will find the NIC of your client device has a new filterdriver to catch traffic destined for GLobal Secure Access. This is the way the client redirects this traffic to the destination (Connector Group, step 1) over the Entra service backplane, as defined defined in your Traffic Forwarding Profile ( step 2). 

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/NIC.png)

## Testing in practice 

The client device I will use to test is a local Hyper-V guest. A windows 11 Enterprise client with TPM enabled, Entra Joined. 

From its local commandline, i have asked it to setup a SSH connection with one of the linux servers in Azure. Obviously there is no network connection such as VPN or ExpressRoute connecting this Virtual Client with my Azure network. 

Instead, the GSA Filterdriver picks up my request to connect to the known IP address and port combination, as defined in the Enterprise App and Trafic Profile and presents an Entra sign-in. Alyx gets a Sign In request, and once completed she can successfully connect to the SSH service in Azure.

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/Client_catch.png)

I found in my case Alyx only had to sign in once, and could leverage SSO from that point on to all GSA Apps.
It seemed to trigger the registration of the user object in the Global Secure Access Client's local profile.

Since the product at this time is still in preview, chances are this behavior may chance.

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/GSAprofile.png)

And there you have it. SSH connectivity, no line of sight, and Entra based authentication. 
As the product currently is in Preview, I would advice to get familiar with the technology right away, so your organisation is ready when GA hits. 
This has such potential, not only as a VPN replacement. And when combining with Entra Internet Access we have a whole new toolset on our belts that enables better availability of services under secure conditions. Can't wait. 





