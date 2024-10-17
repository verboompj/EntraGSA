
###

## Topics 

[Intro of Entra GSA ](https://github.com/verboompj/EntraGSA/blob/main/README.md)

[Part One of Entra GSA - Initial Setup - This Document ](https://github.com/verboompj/EntraGSA/blob/main/EntraGSA_Part1.md)

[Part Two of Entra GSA - SMB over GSA](https://github.com/verboompj/EntraGSA/blob/main/EntraGSA_Part2.md)


## Setup Entra Private Access

The Entra Private Access service consists of 3 main components:
  
1. A Connector - This is a (group of) server(s) that has a line of sight to the service one wants to expose - a web, rds, ssh, whatever service you want your client to be able to connect to. On these servers one installs the Connector Service to reverse-connect into the Entra platform. 

2. An Entra Application registration, representing the service you'd like to expose, the conditional access policy and the user assignment. This is the " traditional" Entra Enterprise Application as we know it + Network Access properties.

3. A Client - Global Secure Access Client - installers available for Windows, Android, IOS and macOS - installed on the client device.

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/private-access-diagram-quick-access3.png)

#### 0. Make sure you meet the prereqs 

Make sure you have admin access to the Entra Admin Center https://entra.microsoft.com 

Have the required P1 licence(s) & assig them to your (test) users.

Requirements may change over time, make sure to keep track using this URL: https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-get-started-with-global-secure-access#prerequisites 



#### 1. Connector Service

Download and install the Connector Service from the Entra Admin portal onto a available (Windows)server.

The connector service/server, in my example, is a Windows server deployed in Azure. It is deployed in the same vnet , however on a different subnet, than the service I'd like to publish/expose using Entra Private access. In my case I'd like to expose a set of Linux Virtual machines over SSH. 

Non of my deployed servers have a public IP address assigned. In this case my Windows server does have outbound Internet connectivity using a NAT gateway. No port forwards have been configured, and only outbound connectivity is allowed.

I want users to be able to connect to these Linux servers over SSH without setting up any form of network connectivity using traditional service such as VPN. Instead, I'd like my users to connect from their local devices to the private IP addresses of these Linux VM's, directly over the internet. In their case any wifi / 5G connection available.

In order to achieve this, my Windows Server is confirmed to have line of sight with the Linux servers, has outbound Internet connectivity and is eqquiped with the (Entra) Connector Service. As soon as the service is installed, the server pops up in the Entra portal as an available connector: 

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/connectors2.png)

Here one can group multiple connectors into a group for redundancy purposes. For production a minimum of 2 servers is recommended. The servers need to share the common basics such as connectivity to the "to be exposed" service(s). 

#### 2. Entra App Registration

On the Entra Admin portal:  To define the exposed service one can create an Enterprise Application, name it, select the Connector Group to be used ( step 2) and under Application Segment, define an Application Segment, where one defines the (private) FQDN or rfc1918 IP-address and the Portnumber of the service. In this case, for SSH, its port 22. 
Other options are a complete CIDR range or a custom IP-to-IP range and Port combination.  

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/EntraAppEnt2.png)

Next, we have to setup a Traffic Forwarding Profile. This enables our configuration of the Entra Private Access Enterprise App to flow to the clients in the field. Select the Private access profile and activate it.

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/forwarding.png)

Thats it, you can visit the Dashboard page for an overview of steps 1 and 2:

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/EntraSummary2.png)

#### 3. Global Secure Access Client

Installing the client is straight forward, there are a few catches, such as the client device having to be either Entra or Hybrid joined (not registered) The requirements are listed here: https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-install-windows-client#prerequisites

Once setup, you will find the NIC of your client device has a new filterdriver to catch traffic destined for Global Secure Access. This is the way the client redirects this traffic to the destination (Connector Group, step 1) over the Entra service backplane, as defined defined in your Traffic Forwarding Profile (step 2). 

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/NIC.png)

## Testing in practice 

The client device I will use to test is a local Hyper-V guest. A windows 11 Enterprise client with TPM enabled, Entra Joined. 

From its local commandline, i have asked it to setup a SSH connection with one of the linux servers in Azure using its private IP address of 10.40.0.4 . Obviously there is no network connection such as VPN or ExpressRoute connecting this Virtual Client with my Azure network. 

Instead, the GSA Filterdriver picks up my request to connect to the known IP address and port combination, as defined in the Enterprise App and Trafic Profile and presents an Entra sign-in. Alyx gets a Sign In request, and once completed she can successfully connect to the SSH service in Azure.

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/Client_catch.png)

I found in my case Alyx only had to sign in once, and could leverage SSO from that point on to all GSA Apps.
It seemed to trigger the registration of the user object in the Global Secure Access Client's local profile:

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/GSAprofile.png)

Since the product at this time is still in preview, chances are this behaviour may chance.


### Concluding

And there you have it. SSH connectivity, no line of sight, and Entra based authentication. 
As the product currently is in Preview, I would advice to get familiar with the technology right away, so your organisation is ready when GA hits. 
This has such potential, not only as a VPN replacement. And when combining with Entra Internet Access we have a whole new toolset on our belts that enables better availability of services under secure conditions. Can't wait. 





