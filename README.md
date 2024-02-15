# Intro into Entra Global Secure Access

Entra GSA is Microsoft's SSE (Security Service Edge) Solution and consists of 2 main services - 
##### Entra Internet Access and Entra Private Access.


![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/global-secure-access-diagram.png)



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

i.e. an Identity (& Device) based upstream Secure Web Gateway as part of the Entra platform, leveraging CASB and Conditional Access components of that very suite.  

### Entra Private Access - 
Microsoft Entra Private Access provides your users - whether in an office or working remotely - secured access to your private, corporate resources. Microsoft Entra Private Access builds on the capabilities of Microsoft Entra application proxy and extends access to any private resource, port, and protocol.

#### Key features
- Quick Access: Zero Trust based access to a range of IP addresses and/or FQDNs without requiring a legacy VPN.
- Per-app access for TCP apps (UDP support in development).
- Modernize legacy app authentication with deep Conditional Access integration.
- Provide a seamless end-user experience by acquiring network traffic from the desktop client and deploying side-by-side with your existing third-party SSE solutions.

i.e. Azure AD ( Entra) App Proxy on Steroids; the VPN killer :-) 



This is a very common scenario, where for instance the Network Team manages the central VNET and all of its sub-components ( Subnets, IP-Adressing, Gateways, ExpressRoutes, etc), all contained within a specific Resource Group that is subjected to the Azure RBAC model. 

Teams deploying services into the subscription would do so in their own respective ResourceGroups, whilst still leveraging the centrally controlled VNET of ResourceGroup "A" , and leveraging all the services available in that VNET.

### Setup 

To create a VM with its NIC connected to a Vnet that lives outside of the ResourceGroup the VM is created in, its prefered to first declare a variable for the actual VNET and SUBNET you want to use, and simply call that variable on deployment of your VM.

Example code : `export SUBNETID="$(az network vnet subnet show --resource-group [ResourceGroup containing VNET] --vnet-name [NAME of VNET] -n [NAME of Subnet]  --query id -o tsv)"`

The example does an export to tsv as variable `$SUBNETID` of the target Vnet and Subnet. It is actually capturing the full ResourceID
You can display the Resource ID by querying the $Subnet 

The results:

![Screenshot](https://github.com/verboompj/Networking/blob/master/Pictures/72subnetid.PNG)

Next, we create the VM and call the variable `SUBNETID` :

`az vm create \`

`  --resource-group mynewrg \`

`  --name myVM18 \`

`  --image UbuntuLTS \`

#### `  --subnet $SUBNETID \`

`  --admin-username azureuser \`

`  --generate-ssh-keys \`

`  --storage-sku Premium_LRS \`

`  --os-disk-size-gb 50 \`

`  --size Standard_F4s_v2 \`

`  --accelerated-networking true \`

![Screenshot](https://github.com/verboompj/Networking/blob/master/Pictures/73vmcreated.png)

result :-) 

