
###

## Setup SMB (Server Message Block) over Entra Private Access

A quick recap of the GSA components: 
![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/private-access-diagram-quick-access3.png)



## Topics 

[Intro of Entra GSA ](https://github.com/verboompj/EntraGSA/blob/main/README.md)

[Part One of Entra GSA - Initial Setup](https://github.com/verboompj/EntraGSA/blob/main/EntraGSA_Part1.md)

[Part Two of Entra GSA - SMB over GSA](https://github.com/verboompj/EntraGSA/blob/main/EntraGSA_Part2.md)  - This Document 


### 1. Why ? 


Good point. SMB is actually just as old as yours truly - It was developed back in good ol' 1983. Great year. 

Even today a lot of companies are still struggling with traditional Fileshares to organize their workflows. Some organizations are simply stuck with it because changing to modern alternatives is considered too costly or disruptive to the business. 
(who wants to be that manager that prioritizes to optimize operations instead of growing the business, right ? yeah. that’s what their predecessors thought too ;-) ..... )

That leaves us with a bit of an old protocol (no modern authentication & cool stuff like Conditional Access, and in practice limited encryption due to backwards compatibility requirements) and old habits of having that all familiar U:\ , P:\ and G:\ drive mappings on our clients. 

As you can tell, I'm a huge fan of SMB Fileshares in Modern Work environments :-) 

The biggest challenge in the post covid world is that we no longer default to the office as our primary work location. 
We could just run SMB over the Internet, however, most of the Internet Service Providers (ISP) block TCP port 445, not allowing SMB directly over their connections. 
So, to extend SMB fileshares into any location requires either a complete corporate network setup, a VPN topology, or worse; a full-blown VDI environment. 


Enter Entra Global Secure Access. Microsoft's answer to an SSE. As covered previously, Global Secure Access (GSA) offers a nifty reversed proxy that allows us to tunnel specific (very specific) applications between a modern endpoint and in this case a legacy fileshare over the Internet. 

Best of all is the wealth of other Entra features like the Conditional Access Framework that are included. In other words - I can set conditions for accessing a legacy share over the Internet. SMB sounds a whole lot better now :-).

#### As an alternative 
I will mention SMB over QUIC - (https://learn.microsoft.com/en-us/azure/storage/files/storage-files-networking-overview#smb-over-quic) QUIC tunnels SMB over port 443 and uses UDP as transport layer. 

It requires a capable endpoint ( Win 11) and a Windows Server 2022 Azure Edition Server to provide the QUIC endpoint. 
Azure files does not natively support QUIC but can of course be headed with a Fileserver that does support QUIC. 

I don't think it is as capable as enabling SMB shares over GSA, specifically when it comes to security & identity features. 

#



### 2. (any)Fileserver over GSA


Just as with my previous post on GSA, it depends on a moden client (Windows 11 in my case) that is at least Hybrid joined (Entra joined in my case):

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/validateentrajoined.png)

And a back-end service called the 'MicrosoftEntraPrivateNetworkConnector', installed on a (preferebly dedicated) server in your corporate LAN environment. 
This server will do an outbound connect into Entra SSE, allowing Entra to connect incoming Application-tunnel requests to the right Connector service(server) in your corporate LAN. 

No port forwards, all outbound connectivity with the Entra SSE in the middle authenticating and authorizing connections. 

#


![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/AVDBMRG.png)

#### Topology
My Connector Server is deployed in an Azure vnet (entra02 in the picture), with an outbound NAT Gateway attached to the vnet for default egress traffic to the Internet. 

In this very vnet, but in a different subnet I have provisioned a Private Endpoint that will allow my vnet to connect to (in this case) an Azure Files share over private (RFC1918) IP connectivity.

Effectively I want to disable public access to the share and only allow connectivity via the private endpoint. 

I also have a Fileserver in a different vnet in Azure, that i peered to this vnet. This allows for connectivity between this Fileserver and my Connector server, so i can use the same Connector server for both shares. 

I will expose 2 fileshares :
- Fileshare from a Windows fileserver called File01.blackmesa.local
- Azure Files share from a Storage account called smbsharebm.file.core.windows.net, exposed through a private endpoint only.

Both will be accessible from the corporate LAN, in my case an Azure vnet, and exposed through GSA. 


#### Azure Files 


The Azure Files share I have configured for Identity based Access through Entra Kerberos. See [this great Doc here](https://learn.microsoft.com/en-us/azure/storage/files/storage-files-identity-auth-hybrid-identities-enable?tabs=azure-portal%2Cregkey)

It allows me to allow hybrid users to access Azure file shares using Kerberos authentication, using Microsoft Entra ID to issue the necessary Kerberos tickets to access the file share with the SMB protocol. 

This in itself is a great feature, and I will happily extend this scenario with GSA and Private Access. Remember though, your users need to be Hybrid users -- Synced from ADDS to Entra. The client device can be Entra Only, and will need that registry setting for Kerberos Ticketing. (it s in the Doc) 


#



![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/setupazurefiles.png)

With that all setup, i created the Private Endpoint for the Azure Files share and setup DNS conditional forwarding for the privatelink.file.core.windows.net zone. Linked the zone to my DNS server vnet and all was well. Pinging/resolving/accessing the fileshare is now done over my own vnet.

See: (https://learn.microsoft.com/en-us/azure/storage/files/storage-files-networking-endpoints?tabs=azure-portal)

I disabled Public access and only allowed connections through the Private Endpoint going forward.



![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/privlinkdns.png)


#


### 3. GSA rules 

With that I can now create a new application in the Entra suite and publish the 2 FQDN's mentioned for the fileshares.
Port is obviously still 445, and protocol remains TCP. Assign the right Connector Group and save.

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/entranap.png)

Back to the client, in my case a Windows 11 VM that is running as a VM inside Hyper-V locally on my laptop. 
In the GSA client we can pull up the Traffic Profile to show what rules it received for tunnelling. In my case its the SSH rule from part 1 of GSA, and my 2 fileshares I just added. 

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/gsaprofile.png)

And yes, it works :-) Performance is quite good as well, i was hovering around 500Mbit or 50 MB per second copying my favourite ISO's to the network shares ;-). 

The user experience is completely transparent, I continue to use my UNC paths like normal and response is instant. 









