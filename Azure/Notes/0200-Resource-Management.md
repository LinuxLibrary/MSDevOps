# Deploying and Configuring Resources

***Creating a Resource Group***
- Identifying the Azure Regions

	```
	> (Get-AzureRmResourceProvider -ProviderNameSpace Microsoft.Compute).Locations | Sort -Unique
	```

- After identifying all the Azure region we can use a region as of our choice to create resources in.

	```
	> $Location = "West US"
	```

- Now to create a resource group we need a Resource Group Name and a Location in which we want to have our resources

	```
	> $RGVNETName = "OpsSharedVNETPSRG"
	> New-AzureRmResourceGroup -Name $RGVNETName -Location $Location
	```

***Managing Azure Virtual Networks***
- Virtual Network Objects
	- Address Prefix(es)
	- Subnet(s)
	- Optional Gateway for hybrid connectivity
- **CmdLets**
	- New-AzureRmVirtualNetwork
		- Create a Virtual Network
	- New-AzureRmVirtualNetworkSubnetConfig
		- Create a subnet configuration
	- New-AzureRmVirtualNetworkGateway
		- Create a new gateway for Virtual Network
- Let us crate a Class-A network `10.0.0.0/16` with 2 subnets for `Apps` and `Data` with a prefix of `/24`. Refer to the below config.

```
> $vnetName = "OpsTrainingVNETPS"
> $addressSpaces = @()
> $addressSpaces += "10.0.0.0/16"
> $subnets = @()
> $subnets += New-AzureRmVirtualNetworkSubnetConfig -Name "Apps" -AddressPrefix 10.0.1.0/24
> $subnets += New-AzureRmVirtualNetworkSubnetConfig -Name "Data" -AddressPrefix 10.0.2.0/24
> $vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $RGVNETName -Location $Location -AddressPrefix $addressSpaces -Subnet $subnets
```

***Creating Azure Virtual Machines***
- Common CmdLets
	- Get-AzureRmVM
	- New-AzureRmVM
	- Update-AzureRmVM
	- Stop-AzureRmVM
	- Start-AzureRmVM

- Typical flow for creating a Virtual Machine
	- Create Resource Group
	
	```
	> $Location = "West US"
	> $rgVMName = "OpsTrainingVMRG"
	> New-AzureRmResourceGroup -Name $rgVMName -Location $Location
	```

	- Create Storage Account

	```
	> $storageAccount = "[Storage Account Name]"
	> New-AzureRmStorageAccount -ResourceGroupName $rgVMName -Location $Location -Name $storageAccount -Type Standard_LRS
	```

	- Create NSG (Network Security Group) (Optional)
	- Create Public IP (Optional)
	- Create Network Interface
	- Create Availability Set (Optional)
	- Set OS Credentials
	- Set the Image