# Taking Network Packet Capture on Azure VNG

Network packet capture allows you to create capture sessions to track traffic to and from a virtual machine. Packet capture helps to diagnose network anomalies, both reactively, and proactively. Other uses include gathering network statistics, gaining information on network intrusions, to debug client-server communication, and much more. Being able to remotely trigger packet captures, eases the burden of running a packet capture manually on a desired virtual machine, which saves valuable time.
In this article, you learn to capture network packet traversing Azure VNG using PowerShell script.
Reference link: GatewayPacketCapture.
If you have the below script (Default) in my earlier post, it came to my discovery that it produces an empty capture within the container after the script ran its course but I was able to fix this problem in my next tweaked script (Modified).

#   -- Default -- 

$rgname = "xxxxxxxx"
$gwname = "xxxx"
$storageAccountName = "xxxxxxxx"
$containerName = "xxxxxxx"

$rg = Get-AzResourceGroup -name $rgname

$storageAccount = New-AzStorageAccount -ResourceGroupName $rgname -Name $storageAccountName -Location $rg.Location -SkuName "Standard_LRS"
$key = Get-AzStorageAccountKey -ResourceGroupName $rgname -Name $storageAccountName
$context = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $key[0].Value
$container = New-AzStorageContainer -Name $containerName -Context $context
$sasurl = New-AzStorageContainerSASToken -Name $containerName -Context $context -Permission "rwd" -StartTime (Get-Date).AddHours(-1) -ExpiryTime (Get-Date).AddDays(1) -FullUri

#Start Network Capture
Start-AzVirtualNetworkGatewayPacketCapture -Name $gwname -ResourceGroupName $rgname -FilterData $a

#Stop Network Capture
Stop-AzVirtualNetworkGatewayPacketCapture -Name $gwname -ResourceGroupName $rgname -SasUrl $sasurl 

#   -- Modified -- -

$rgname = "xxxxxxxx"
$gwname = "xxxx"
$storageAccountName = "xxxxxxxx"
$containerName = "xxxxxxx"

$rg = Get-AzResourceGroup -name $rgname

$storageAccount = New-AzStorageAccount -ResourceGroupName $rgname -Name $storageAccountName -Location $rg.Location -SkuName "Standard_LRS"
$key = Get-AzStorageAccountKey -ResourceGroupName $rgname -Name $storageAccountName
$context = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $key[0].Value
$container = New-AzStorageContainer -Name $containerName -Context $context
$sasurl = New-AzStorageContainerSASToken -Name $containerName -Context $context -Permission "rwd" -StartTime (Get-Date).AddHours(-1) -ExpiryTime (Get-Date).AddDays(1) -FullUri

$a = "{`"TracingFlags`": 11,`"MaxPacketBufferSize`": 120,`"MaxFileSize`": 500,`"Filters`" :[{`"CaptureSingleDirectionTrafficOnly`": true}]}"

#Start Network Capture
Start-AzVirtualNetworkGatewayPacketCapture -Name $gwname -ResourceGroupName $rgname -FilterData $a

#Stop Network Capture
Stop-AzVirtualNetworkGatewayPacketCapture -Name $gwname -ResourceGroupName $rgname -SasUrl $sasurl


## Practical Example:

My environment consisted of a hybrid site-to-site connection between an on-premise and Azure gateways. I provided a test machine each behind these gateways basically to send ICMP packet simultaneously across the VPN tunnel. I ran the two scripts separately. 

# Result produced by the “Default Script”:
As seen in the snippet, only ESP traffic 

https://user-images.githubusercontent.com/40485440/92325187-80574e80-f040-11ea-9227-f9977d6c5b78.png

# Result produced by the “Modified Script”:
Snippet below shows ICMP traffic being sent across the tunnel. 

https://user-images.githubusercontent.com/40485440/92325198-96650f00-f040-11ea-89c4-ff904c7e30fc.png

Note:
•	This script is prepared to run on Azure CLI.
•	It is advisable to run the captures for at least 600 seconds.
•	The script creates a storage account, a container within the storage account hosts both capture files (each for the VNG instances
