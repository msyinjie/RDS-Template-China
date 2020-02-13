# Create Remote Desktop Sesson Collection deployment in Azure

This template deploys the following resources:

<ul><li>public IP, load balancer, availabilitySet;</li><li>RD Gateway/RD Web Access vm;</li><li>RD Connection Broker/RD Licensing Server vm;</li><li>a number of RD Session hosts (number defined by 'numberOfRdshInstances' parameter)</li></ul>

The template will use existing DC, join all vms to the domain and configure RDS roles in the deployment.

Click the button below to deploy to Azure China

<a href="https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmsyinjie%2FRDS-Template-China%2Fmaster%2Frds-deployment-existing-ad%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

Click the button below to deploy to Azure Global

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmsyinjie%2FRDS-Template-China%2Fmaster%2Frds-deployment-existing-ad%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

### Prerequisites

Current Template is an extension to the Basic RDS Deployment Template, and it is mandatory to deploy any one of the template as prerequisite:

* Basic RDS deployment prerequisite.  

* RDS deployment on pre-existing VNET and AD (**Notice: The RDS deploy to resource location MUST same as the pre-existing VNET containing the AD**). 

* The DNS server IP which can resolve the AD Domain name.

### Documentation
[Basic RDS farm Deployment](https://azure.microsoft.com/en-us/documentation/templates/rds-deployment/) contains additional information about this template.

[Azure Resource Naming Conventions](https://docs.microsoft.com/en-us/azure/architecture/best-practices/naming-conventions) contains naming rules and restrictions for Azure resource conventsions.

### Base parameters:

* Location - location for new deployment. 
    * PowerShell enumeration: ```Get-AzureRmLocation```
* ResourceGroup - name of new or existing resource group for RDS deployment. 
    * PowerShell new resource group: ```New-AzureRmResourceGroup -Location $location```
* ResourceGroupDeployment - name of new or existing resource group deployment. 
    * PowerShell new deployment: ```New-AzureRmResourceGroupDeployment -Location $location <...>```
 
* adDomainName - active directory domain name. Limited to netbios naming conventions (15 alphanumeric characters).
* adminPassword - password for administrator account both locally and in domain.
    * The supplied password must be between 12-123 characters long and must satisfy at least 3 of password complexity requirements from the following: 
        * Contains an uppercase character
        * Contains a lowercase character
        * Contains a numeric digit
        * Contains a special character.
* adminUsername - Active Directory and local administrator account name. User name conforms to Active Directory user name convention.
* VNET - VNET name of pre-existing containing the AD Domain.
* Subnet - Subnet name of pre-existing containing the AD Domain.
* VNET ResourceGroup - The resource group name of pre-existing containing the VNET containing the AD Domain controler.
* dnsLabelPrefix -  DNS name which is external name used to connect to environment. example name 'gateway.contoso.com' would be 'gateway'. See [Naming conventions in Active Directory](https://support.microsoft.com/en-us/help/909264/naming-conventions-in-active-directory-for-computers,-domains,-sites,-and-ous)
* imageSKU - operating system version for all instances. '2016-Datacenter' or '2019-Datacenter',the default is '2019-Datacenter'.
    * PowerShell enumeration: ```Get-AzureRmVMImageSku -Location $location -PublisherName MicrosoftWindowsServer -Offer WindowsServer```
* numberOfRdshInstances - number of RDS host servers to deploy. Other instances gateway, and broker are set to 1 instance.
* rdshVmSize - virtual machine size for the RDS host server instances only. Other instances gateway, and broker are set to size Standard_D4_v3. 
    * PowerShell enumeration: ```Get-AzureRmVMSize -Location $location```
* Data disk attached the RDSH server
    * Number per VM - Data disk number attached the per RDSH server, the value will be 1 to 4, the default is 2.
    * Disk size - Data disk size per disk, the value will be 1 to 1023, the defaut is 128GB.

### Post Deployment
After deployment completes successfully, use one of the options below to connect to the new deployment.
After deploying this template, the following will need to be configured:

* Remote Desktop Services Licensing (there is an initial 120 day grace period). 
    * See [License your RDS deployment with client access licenses](https://technet.microsoft.com/en-us/windows-server-docs/compute/remote-desktop-services/rds-client-access-license) for information on licensing and steps to configure.
* Trusted certificates for connectivity and RDS infrastructure
* User Profile Disks (not support current)

### Connect to new deployment
After successful deployment, the URL for the Remote Desktop Gateway (RDGW) and RDWeb site will be https://%dnsLabelPrefix%.%location%.cloudapp.azure.com/RDWeb or https://%dnsLabelPrefix%.%location%.cloudapp.chinacloudapi.cn/RDweb. A self-signed certificate will be used for the deployment. To prevent certificate mismatch issues when connecting using a self-signed certificate, the certificate will need to be installed on the local client machines 'Trusted Root' certificate store. Best practice for a production environment is to configure the deployment to use a trusted certificate.

**To install the self-signed certificate into local machine 'Trusted Root' certificate store:**
1. From administrative Internet Explorer, browse to URL.
   * Example from above: 'https://contoso.westus.cloudapp.azure.com'
2. In address bar, click on 'Certificate Error' -> 'View Certificates'
3. Select 'Install Certificate...'
    * certificate name should be in the format of 'CN=%dnsLabelPrefix%.%location%.cloudapp.azure.com'
    * Example from above: 'CN=contoso.westus.cloudapp.azure.com'
6. Store location is 'Local Machine'
7. Select 'Browse' for 'Place all certificates in the following store'
8. Select 'Trusted Root Certification Authorities'
9. After the certificate has been installed, in RDWeb, logon with the domain credentials that were configured when deploying template.
10. Launch 'Desktop Collection'

### Troubleshooting
If the deployment did not complete successfully or if you are having issues connecting to the environment, use one of the options below:

#### Review deployment events in Azure portal
In [Azure Portal](https://portal.azure.com) or [Azure China Portal](https://portal.azure.cn), select the 'Resource Group' containing the deployment. Select 'Overview' if not selected, and the 'Deployments' summary link will be displayed. Selecting the link will navigate to the individual deployments and associated events.

#### Review deployment events with PowerShell
In PowerShell, the following azurerm module commandlets can be used to gather deployment events:
- Get-AzureRmLog 
- Get-AzureRmResourceGroupDeployment
- Get-AzureRmResourceGroupDeploymentOperation


