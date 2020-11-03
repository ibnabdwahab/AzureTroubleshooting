*******************************************************************************************

                    Taking Network Packet Capture on Azure VNG
          
*******************************************************************************************

Network packet capture allows you to create capture sessions to track traffic to and from a virtual machine. Packet capture helps to diagnose network anomalies, both reactively, and proactively. Other uses include gathering network statistics, gaining information on network intrusions, to debug client-server communication, and much more. Being able to remotely trigger packet captures, eases the burden of running a packet capture manually on a desired virtual machine, which saves valuable time. In this article, you learn to capture network packet traversing Azure VNG using PowerShell script.

Reference link: https://docs.microsoft.com/en-us/powershell/module/az.network/start-azvirtualnetworkgatewaypacketcapture?view=azps-4.6.1#example-2

If you have the below script (Default) in my earlier post, it came to my discovery that it produces an empty capture within the container after the script ran its course but I was able to fix this problem in my next tweaked script (Modified).

**************************
       
       Default
       
**************************

Default script

********************************************************************************************************************************************************************************

$rgname = "xxxxxxxx" $gwname = "xxxx" $storageAccountName = "xxxxxxxx" $containerName = "xxxxxxx"

$rg = Get-AzResourceGroup -name $rgname $storageAccount = New-AzStorageAccount -ResourceGroupName $rgname -Name $storageAccountName -Location 
$rg.Location -SkuName "Standard_LRS" 
$key = Get-AzStorageAccountKey -ResourceGroupName $rgname -Name $storageAccountName 
$context = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $key[0].Value 
$container = New-AzStorageContainer -Name $containerName -Context $context 
$sasurl = New-AzStorageContainerSASToken -Name $containerName -Context $context -Permission "rwd" -StartTime (Get-Date).AddHours(-1) -ExpiryTime (Get-Date).AddDays(1) -FullUri

#Start Network Capture Start-AzVirtualNetworkGatewayPacketCapture -Name $gwname -ResourceGroupName $rgname -FilterData $a

#Stop Network Capture Stop-AzVirtualNetworkGatewayPacketCapture -Name $gwname -ResourceGroupName $rgname -SasUrl $sasurl

********************************************************************************************************************************************************************************
