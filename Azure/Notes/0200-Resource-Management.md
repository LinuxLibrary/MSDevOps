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
	- **Create Resource Group**
	
	```
	> $Location = "West US"
	> $rgVMName = "OpsTrainingVMRG"
	> New-AzureRmResourceGroup -Name $rgVMName -Location $Location
	```

	- **Create Storage Account**

	```
	> $storageAccount = "[Storage Account Name]"
	> New-AzureRmStorageAccount -ResourceGroupName $rgVMName -Location $Location -Name $storageAccount -Type Standard_LRS
	
	Other supported types:
		- Standard_LRS	: Locally Redundant Storage
		- Standard_GRS	: Geo-Redundant
		- Standard_GARS	: Geo-Redundant Read Access
		- Premium_LRS	: Premium Locally Redundant
		- Standard_ZRS	: Zero Redundant (Not supported for VMs)
	```

	- **Create NSG (Network Security Group) (Optional)**

	```
	> $rules = @()
	> $rules += New-AzureRmNetworkSecurityRuleConfig -Name "RDP" -Protocol Tcp -SourcePortRange "*" -DestinationPortRange "3389" -SourceAddressPrefix "*" -DestinationAddressPrefix "*" -Access Allow -Description "Allow RDP Access" -Priority 100 -Direction Inbound
	> $nsg = New-AzureRmNetworkSecurityGroup -Name "webnsg" -ResourceGroupName $RGVNETName -Location $Location -SecurityRules $rules
	```

	- **Create Public IP (Optional)**

	```
	# Check for unique DNS name
	> Test-AzureRmDNSAvailability -DomainQualifiedName "[unique DNS name]" -Location $Location
	> $dnsName = "[unique DNS name]"
	> $ipName = "webVMPubIP"
	> $pip = New-AzureRmPublicIpAddress -Name $ipName -ResourceGroupName $RGVNETName -Location $Location -AllocationMethod Dynamic -DomainNameLabel $dnsName

	AllocationMethod:
		- Dynamic assigned to a Network Interface or Load Balancer
		- Static only when assigned to a Load Balancer
	```

	- **Create Network Interface**
	
	```
	> $vnet = Get-AzureRmVirtualNetwork -ResourceGroupName $rgVMName -Name VNETName
	> $nicName = "webVMNIC1"
	> $nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgVMName -Location $Location -SubnetId $vnet.subnets[0].Id -PublicAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id
	
	* -PublicIpAddressId - associates with a Public IP (optional)
	* -SubnetId - associates with the virtual network and subnet
	* -NetworkSecurityGroupId - associates with a network security group (optional)
	```

	- **Create Availability Set (Optional)**

	```
	> $avSet = New-AzureRmAvailabilitySet -ResourceGroupName $rgVMName -Name "webAVSET" -Location $Location -PlatformUpdateDomainCount 5 -PlatformFaultDomainCount 3
	
	* -PlatformUpdateDomainCount - default 5, maximum of 20
	* -PlatformFaultDomainCount - default 3, maximum of 3
	```

	- ***Create VM Configuration***

	```
	> $vmName = "webvm-1"
	> $vmSize = "Standard_A1"
	> $vm = New-AzureRmVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avSet.Id

	* New-AzureRmVMConfig creates a local object that can be configured with settings such as NICs, Storage, etc.,
	* The availability set is specified by specifying Id of the previously created Availability Set to the -AvailabilitySetId parameter
	```

	- ***Attaching a Data Disk***
	
	```
	# Get the current storage account
	> $storageAcc = Get-AzureRmStorageAccount -ResourceGroupName $rgVMName -Name $storageAccount

	# Return the HTTP endpoint
	> $blobEndpoint = $storageAcc.PrimaryEndpoints.Blob.ToString()
	> $dataDisk1Name = "vm1-datadisk1"
	
	# Build the Full url to VHD
	> $dataDisk1Uri = $blobEndpoint + "vhds/" + $dataDisk1Name + ".vhd"
	
	# Specify the disk on the VM configuration
	> $vm | Add-AzureRmVMDataDisk -Name "datadisk1" -VhdUri $dataDisk1Uri -Caching None -DiskSizeInGB 1023 -Lun 0 -CreateOption empty
	
	* The Add-AzureRmVMDataDisk cmdlet modifies the virtual machine configuration object ($vm)
	* $vm can be piped in as shown in the example or passed via the -VM parameter
	```

	- **Set OS Type and Credentials**
	
	```
	> $cred = Get-Credential -Message "Enter Admin Credentials"
	> $vm | Set-AzureRmVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred - ProvisionVMAgent
	
	* Coming from Classic - this is similar to the Add-AzureProvisioningConfig cmdlet
	* It does not support the -WindowsDomain parameter set though
	* Supports -Linux parameter set
	* Supports -CustomData patameter set to specify a custom file to be passed to the operating system
	```

	- **Set the Image**

	```
	> $pubName = "MicrosoftWindowsServer"
	> $offerName = "WindowsServer"
	> $skuName = "2012-R2-Datacenter"
	> $vm | Set-AzureRmVMSourceImage -PublisherName $pubName -Offer $offerName -Skus $skuName -Version "latest"
	> $osDiskName = "vm1-osdisk0"
	> $osDiskUri = $blobEndpoint + "vhds/" + $osDiskName + ".vhd"
	> $mv | Set-AzureRmVMOSDisk -Name $osDiskName -VhdUri $osDiskUri -CreateOption fromImage
	```
