# 如何创建DS系列虚拟机 #

> 本文说明如何使用Azure PowerShell和 Azure Cli 创建DS系列虚拟机。

--------

## 使用powershell创建DS系列虚拟机 ##


### 创建高级存储账户 ###

	`New-AzureStorageAccount -StorageAccountName "yourpremiumaccount" -Location "China East" -Type "Premium_LRS"`
	
    
> - 创建高级存储帐户时，必须将 Type 参数指定为 Premium_LRS

> - 目前为止，尽在中国东部支持高级存储账号

### 指定订阅及高级存储账户 ###

	`Set-AzureSubscription -SubscriptionName "your subscription"  -CurrentStorageAccountName "yourpremiumaccount"`
	
### 创建DS系列虚拟机 ###
    
	 $storageAccount = "yourpremiumaccount"
	 $adminName = "youradmin"
	 $adminPassword = "yourpassword"
	 $vmName ="yourVM"
	 $location = "China East"
     $imageName = " 55bc2b193643443bb879a78bda516fc8__Windows-Server-2012-Datacenter-201504.01-zh.cn-127GB.vhd "
     $vmSize ="Standard_DS2"
	 $OSDiskPath = "https://" + $storageAccount + ".blob.core.chinacloudapi.cn/vhds/" + $vmName + "_OS_PIO.vhd"
	 $vm = New-AzureVMConfig -Name $vmName -ImageName $imageName -InstanceSize $vmSize -MediaLocation $OSDiskPath   
	 Add-AzureProvisioningConfig -Windows -VM $vm -AdminUsername $adminName -Password $adminPassword 
	 New-AzureVM -ServiceName $vmName -VMs $VM -Location $location

> 如何获取当前可用的ImageName（由于Azure会定期更新ImageName，为了保证您能够填写正确的ImageName）可以通过下面的脚本查询一下。比如：
>
    #操作系统类型OpenLogic 6.5，Windows Server 2012 R2 Datacenter (zh-cn)等
	$osfamliy="Windows Server 2012 R2 Datacenter (zh-cn)"  
	Get-AzureVMImage | Where-Object {
	$_.category -eq "Public" -and 
	($_.ImageFamily -eq $osfamliy -or $_.Label -eq $osfamliy )}| Sort-Object publisheddate -Descending|
	select imagename,os,label,publisheddate -First 1|
	Format-Table -AutoSize
>   ![](http://i.imgur.com/wNP3fCt.png)
### 为虚拟机附加磁盘 ###

	$storageAccount = "yourpremiumaccount"
	$vmName ="yourVM"
	$vm = Get-AzureVM -ServiceName $vmName -Name $vmName
	$LunNo = 1	
	$path = "http://" + $storageAccount + ".blob.core.chinacloudapi.cn/vhds/" + "myDataDisk_" + $LunNo + "_PIO.vhd"
	$label = "Disk " + $LunNo
	Add-AzureDataDisk -CreateNew -MediaLocation $path -DiskSizeInGB 128 -DiskLabel $label -LUN 
	$LunNo -HostCaching ReadOnly -VM $vm | Update-AzureVm


## 使用Azure Cli 创建DS系列虚拟机 ##

### 创建高级存储账号 ###

    azure storage account create "premiumtestaccount" -l "china east" --type PLRS

### 创建DS系列虚拟机　###

	azure vm create -z "Standard_DS2" -l "china east" -e 22 
	"premium-test-vm" "f1179221e23b4dbb89e39d70e5bc9e72__OpenLogic-CentOS-70-20150325" 
	--userName "youradminname" 
	-p "Passwd@123"

>  CentOS 7.0 为例

### 挂载数据磁盘 ###

	azure vm disk attach-new premium-test-vm 128 "https://premiumtestaccount.blob.core.chinacloudapi.cn/vhds/myDataDisk_1_PIO.vhd"