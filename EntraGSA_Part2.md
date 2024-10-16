
###

## Setup SMB (Server Mesage Block) over Entra Private Access

A quick 
![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/private-access-diagram-quick-access3.png)

#### 0. Make sure you meet the prerqs 

Make sure you have admin access to the Entra Admin Center https://entra.microsoft.com 

Have the required P1 licen
Requirements may change over time, make sure to keep track using this URL: https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-get-started-with-global-secure-access#prerequisites 



#### 1. Connector Service

Download and install the Connector Service from the Entra Admin portal onto a available (Windows)server.
rs, has outbound Internet connectivity and is equiped with the (Entra) Connector Service. As soon as the service is installed, the server pops up in the Entra portal as an available connector: 

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/connectors2.png)

