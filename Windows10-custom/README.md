# Create Windows 10 Collection deployment in Azure using custom image

This template deploys the following resources:

<ul><li>public IP;</li><li>a number of Windows 10 hosts (number defined by 'numberOfInstances' parameter)</li></ul>

The template will use existing DC, join all vms to the domain.

Click the button below to deploy to Azure China

<a href="https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmsyinjie%2FRDS-Template-China%2Fmaster%2FWindows10-custom%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

Click the button below to deploy to Azure Global

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmsyinjie%2FRDS-Template-China%2Fmaster%2FWindows10-custom%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

### Prerequisites

Current Template is an extension to the Basic VM Deployment Template, and it is mandatory to deploy any one of the template as prerequisite:

* Deployment on pre-existing VNET and AD (**Notice: The deploy to resource location MUST same as the pre-existing VNET containing the AD**). 

* The DNS server IP which can resolve the AD Domain name.

### Documentation
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
* numberOfInstances - number of Windows 10 host to deploy. the max instance limit to 99.the default is 2.
* VmSize - virtual machine size for the Windows 10 host instances. the default is **Standard_D4_v3**. 
    * PowerShell enumeration: ```Get-AzureRmVMSize -Location $location```
* TemplateImageUri - URI for the template VHD to use for host instances. For example, https://rdsstorge.blob.core.chinacloudapi.cn/vhds/Windows10Image.vhd, this image **MUST** based on Windows 10 and **MUST** be Syspreped.

### Connect to new deployment
After successful deployment, the URL for the Remote Desktop will be %dnsLabelPrefix%-XX.%location%.cloudapp.chinacloudapi.cn(XX is 01-99). 

### Troubleshooting
If the deployment did not complete successfully or if you are having issues connecting to the environment, use one of the options below:

#### Review deployment events in Azure portal
In [Azure Portal](https://portal.azure.com) or [Azure China Portal](https://portal.azure.cn), select the 'Resource Group' containing the deployment. Select 'Overview' if not selected, and the 'Deployments' summary link will be displayed. Selecting the link will navigate to the individual deployments and associated events.

#### Review deployment events with PowerShell
In PowerShell, the following azurerm module commandlets can be used to gather deployment events:
- Get-AzureRmLog 
- Get-AzureRmResourceGroupDeployment
- Get-AzureRmResourceGroupDeploymentOperation