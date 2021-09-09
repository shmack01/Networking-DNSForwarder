# Networking-DNSForwarder

## Scenario

There are two possible approaches:

**Private Link** - DNS lookup will default to the public endpoint IP address when a query is not originated from within the VNet. [See Docs](https://docs.microsoft.com/en-us/azure/storage/files/storage-files-networking-dns#configuring-dns-forwarding) For Private Link resources, they require an on-premises DNS server with a conditional forwarder configured to the private IP address of the Azure DNS. For environments that require a CNAME record, this IP-Based solution is not applicable. 

**Private IP** - Queries must originate from the Virtual Network to Azure DNS to resolve to a private endpoint. [See Docs](https://docs.microsoft.com/en-us/azure/private-link/private-endpoint-dns#on-premises-workloads-using-a-dns-forwarder). To serve on-premises requests, a DNS forwarder in Azure is required to resolve the FQDN to a private IP. 

**Environment**
- VNET(s) with one subnet specifically for App Gateway, 1 subnet for private endpoints, and subnets for other resources.
- VPN Gateway
- App Gateway, VM for DNS Forwarder, Privatelink.* Private DNS Zones to A Record for resources. 

mycompany.com Private DNS Zone configuration:
- Linked to Vnet(s). 
- **"Virtual network links"** are Auto-Registration. 

![VNET Link](./images/dnslink.jpg)

### DNS Forwarder
Create Windows Server VM for DNS Forwarder in VNET with Private DNS Zone. Optional: There is also a [Linux solution](https://github.com/Azure/azure-quickstart-templates/tree/master/demos/dns-forwarder). The Private IP will automatically register as an A Record in Private DNS Zone because of the Auto-Registration. 

**VM Configuration**:
- Log into Windows VM and execute command(This can also be installed by custom script extension): 
```	
    Install-WindowsFeature -Name DNS -IncludeManagementTools
```
- Navigate to the DNS Manager -> Right Click Name -> Properties and add Azure Private DNS IP address.


![DNS Forwarder Configuration ](./images/dnsforwarder.jpg)

- Add DNS Forwarder private IP to DNS servers to the VNet(s).  

![DNS Forwarder IP VNET](./images/dnslink.jpg)

- Restart all servers and download a new VPN device. 

### On-Prem machine
- Clear-DnsClientCache or ipconfig /flushdns
- Resolve-DnsName -Name ftagovdnsproxy.mycompany.com

This should resolve to the private IP address. 

###Access to Private Link Resources

Deploy the App Gateway in the VNet. Add an A Record in the private DNS Zone. e.g. appgw.mycompany.com

![Private DNS Zone](./images/privatedns.jpg)


Set App Gateway to point to your App Service for the backend. The App Gateway backend is the FQDN of the App Service, which is the Private Endpoint.  The configuration below is port 80 for the Frontend Listener, which has the rule to the port 443 backend for the App Service. 

![Application Gateway](./images/appgw.jpg)


## Resources

https://docs.microsoft.com/en-us/answers/questions/474171/azure-private-dns-zone-child-zone-from-on-prem-dns.html <br/>
https://docs.microsoft.com/en-us/azure/private-link/private-endpoint-dns <br/>
