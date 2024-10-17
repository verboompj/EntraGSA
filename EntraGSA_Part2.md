
###

## Setup SMB (Server Mesage Block) over Entra Private Access

A quick recap of hte GSA components: 
![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/private-access-diagram-quick-access3.png)

#### Part 1 for the basics

[Part One of Entra GSA - Initial Setup](https://github.com/verboompj/EntraGSA/blob/main/EntraGSA_Part1.md)

Part 2 will focus on extending the usecase to running SMB over GSA. 

#### 1. Why ? 


Good point. SMB is actually just as old as yours truly - It was developed back in good ol' 1983. Great year. 

Even today a lot of companies are still struggling with traditional Fileshares to organize their workflows. Some organizations are simply stuck with it because changing to modern alternatives is considered too costly or disruptive to the business. Who wants to be that manager that prioritizes to optimize operations instead of growing the business, right ? yeah. that’s what your predecessors thought too ;-) ..... 

That leaves us with a bit of an old protocol (no modern authentication & cool stuff like Conditional Access, and in practice limited encryption due to backwards compatibility requirements) and old habits of having that all familiar U:\ , P:\ and G:\ drive mappings on our clients. 

As you can tell, I'm a huge fan of SMB Fileshares in Modern Work environments :-) 

The biggest challenge in the post covid world is that we no longer default to the office as our primary work location. And to extend SMB fileshares into any location either requires a complete corporate network setup, a VPN topology, or worse; a full-blown VDI environment.

Enter Entra Global Secure Access. Microsoft's answer to a SSE. As covered previously, Global Secure Access (GSA) offers a nifty reversed proxy that allows us to tunnel specific (very specific) applications between a modern endpoint and in this case a legacy Fileshare over the Internet. Best of all is the wealth of other Entra features like the Conditional Access Framework that are included. In other words - I can set conditions for accessing a legacy share over the Internet. SMB sounds a whole lot better now :-).


### 2. (any)Fileserver over GSA


Just as with my previous post on GSA, it depends on a moden client (Windows 11 in my case) that is at least Hybrid joined (Entra joined in my case):

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/validateentrajoined.png))

And a back-end service called the 'MicrosoftEntraPrivateNetworkConnector', installed on a (preferebly dedicated) server in your corporate LAN environment. This server will do an outbound connect into Entra SSE, allowing Entra to connect incomming Application-tunnel requests to the right Connector service in your LAN. No port forwards, all outbound connectivity with the Entra SSE in the middle authenticating and authorizing connections. 


