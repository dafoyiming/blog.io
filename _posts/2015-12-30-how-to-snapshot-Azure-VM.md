# Azure 虚拟机镜像管理脚本说明 #

[http://blogs.technet.com/b/canitpro/archive/2014/12/11/step-by-step-create-a-vm-snapshot-in-azure.aspx](http://blogs.technet.com/b/canitpro/archive/2014/12/11/step-by-step-create-a-vm-snapshot-in-azure.aspx "step-by-step-create-a-vm-snapshot-in-azure")

----------

## 准备工作 ##
	
1. 了解文件结构
   ![](http://i.imgur.com/i2hJeiW.png)
2. 导入PublishSetting File
		Get-AzurePublishSettingsFile
		Import-AzurePublishSettingsFile
3. 填写csv文件获取指纹信息
   ![](http://i.imgur.com/UzUm7Sz.png)
   ![](http://i.imgur.com/sRgEkey.png)
## 主要问题 ##

- 脚本编写于2013年，因此对Powershell的版本有一定的要求，需要使用较早版本的Powershell（推荐使用Powershell 0.8.12）
- 脚本针对Global Azure 设计，有些代码需要针对China Azure进行修改

## 修改点 ##

1. 修改Azure Import-Module 为 

   Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
			
			- SnapshotVirtualMachine.ps1    51行
			- RestoreVirtualMachine.ps1     48行
			- GetSnapshotList.ps1           43行
			- DeleteOldSnapshots.ps1        37行
			- Commom/RepositoryCommon.ps1   13行
		
2. 修改Common/RepositoryCommon

- 116行
         
添加-Environment AzureChinaCloud
         		
	Set-AzureSubscription -Environment AzureChinaCloud -Certificate $managementCertificate -SubscriptionId $subscriptionId -CurrentStorageAccount $storageAccount -SubscriptionName $subscriptionName -SubscriptionDataFile $subscriptionDataFile

- 120行
	    
添加-Environment AzureChinaCloud
         
	Set-AzureSubscription -Environment AzureChinaCloud -Certificate $managementCertificate -SubscriptionId $subscriptionId -CurrentStorageAccount $storageAccount -SubscriptionName $subscriptionName -SubscriptionDataFile $subscriptionDataFile
        
- 258行
	    
添加EndpointSuffix=core.chinacloudapi.cn;
				 
	$connectionString = "DefaultEndpointsProtocol=$($protocol);AccountName=$($storageAccountName);AccountKey=$($primaryKey);EndpointSuffix=core.chinacloudapi.cn;"
        
- 291行
        
原代码 .core\.windows\.net
               	
	$matchList = [System.Text.RegularExpressions.Regex]::Match($blobUri, "^(?<Protocol>http[s]?)://(?<StorageAccountName>[^/]*)\.blob\.core\.windows\.net/(?<ContainerName>[^/]*)/(?<BlobName>.*)$", $regexOptionFlags)
        
新代码 .core\.chinacloudapi\.cn
              	 
	$matchList = [System.Text.RegularExpressions.Regex]::Match($blobUri, "^(?<Protocol>http[s]?)://(?<StorageAccountName>[^/]*)\.blob\.core\.chinacloudapi\.cn/(?<ContainerName>[^/]*)/(?<BlobName>.*)$", $regexOptionFlags)
                                                                                      
## 脚本执行 ##

1. 创建虚拟机镜像
		PS C:\Scripts> .\SnapshotVirtualMachine.ps1 -subscriptionName Internal-002 -cloudServiceName zymub -virtualMachineName zymub -shutdownMachine -snapshotOsDisk  -snapshotDataDisks
   ![](http://i.imgur.com/1IsxOO9.png)
2. 列出虚拟机已有的镜像
	   
		PS C:\Scripts> .\GetSnapshotList.ps1 -subscriptionName Internal-002 -cloudServiceName zymub -virtualMachineName zymub -maximumDays 15
   ![](http://i.imgur.com/6o3BQx1.png)
3. 清除虚拟机镜像
	   
		PS C:\Scripts> .\DeleteOldSnapshots.ps1 -subscriptionName Internal-002 -cloudServiceName zymub -virtualMachineName zymub -maximumDays 1
   ![](http://i.imgur.com/U7PtugE.png)
4. 还原虚拟机
	   
		PS C:\Scripts> .\RestoreVirtualMachine.ps1 -subscriptionName Internal-002 -cloudServiceName zymub -virtualMachineName zymub -utcRestoreDate "2015-DEC-26 10:05:43"
   ![](http://i.imgur.com/BJPtCSC.png)


----------
EOF





