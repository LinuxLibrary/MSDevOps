# Installation of Azure Powershell

- Install via Web Platform Installer (Web PI)
- Direct install link (kicks off Web PI)

	```
	http://aka.ms/webpi-azps
	```

- With PowerShell 5 install directly

	```
	> Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
	> Install-Module AzureRM
	> Install-AzureRM
	> Import-AzureRM
	> Import-Module Azure
	```

***Authentication/Authorization***
- Add-AzureAccount (ASM)
	- Login using classic (ASM) mode
	- Still needed for some cmdlets like Test-AzureName
- Login-AzureRmAccount (ARM)
	- Interactive mode to dialog to login
	- Supports -Credentials parameter
	- Supports using a service principal (Azure AD or certificate based) for automated logins for services and automation accounts

***Basics of Subscription Management***
- Get-AzureRmSubscription
	- Enumerates subscriptions you have access to
- Get-AzureRmContext
	- Returns the current ARM context PowerShell is executing
- Set-AzureRmContext
	- Sets the Authentication information for the session
	- Aliased to Select-RmSubscription
	- Supports
		- SubscriptionName
		- SubscriptionId
		- TenantId
		- Context

# Getting Started with Azure Powershell

- Open the PowerShell ISE
- To get the list of Azure commands we can use the `Get-Command` cmdlet

	```
	> Get-Command | Where Name -Like "*Azure*"
	```

- To login to the Azure account use the following cmdlet and provide the credentials

	```
	> Login-AzureRmAccount
	```
- To get a list of Azure subscriptions we have access to 

	```
	> Get-AzureRmSubscription
	```

- To work with a specific subscription we need to set it using either SubscriptionName or SubscriptionId

	```
	> Set-AzureRmContext -SubscriptionId <SUBSCRIPTION_ID_HERE>
	```

- To know the locations

	```
	> (Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Compute).Locations | Sort -Unique
	```

- To create a Resource Group, we need the location and name of the resource group

	```
	> $Location = "West US"
	> $RGVNETName = "OpsSharedVNETPSRG"
	> New-AzureRmResourceGroup -Name $RGVNETName -Location $Location
	```
