# 关于Azure云服务部署多VIP #


### 部署架构 ###



![](http://i.imgur.com/T6lVMkb.png)

> 使用Azure官方的一个架构图,该架构基于SSL多站点服务的设计理念。此文中我们只实现多个VIP部署及终结点管理相关的实验

### 为云服务部署多VIP ###

1. 设置Azure基础环境
        
    	
		$serviceName = 'zymvips'
		$storageAccountName = 'zymvips'
		$location="China North"
        
		#设置当前订阅环境
		$subscription = Get-AzureSubscription -Current
		New-AzureStorageAccount -StorageAccountName $storageAccountName -Location $location
		Set-AzureSubscription -SubscriptionId $subscription.SubscriptionId -CurrentStorageAccountName $storageAccountName  
    	
1. 创建一个云服务及虚拟机（此操作也可以在Azure管理portal中完成）
		
		#查询当前可用的Windows Server 2012系统镜像
		$osfamliy="Windows Server Essentials Experience on Windows Server 2012 R2 (zh-cn)"
		Get-AzureVMImage | Where-Object {$_.category -eq "Public" -and 
		($_.ImageFamily -eq $osfamliy -or $_.Label -eq $osfamliy )}| Sort-Object publisheddate -Descending|
		select imagename,os,label,publisheddate -First 1|Format-Table -AutoSize 

	
	![](http://i.imgur.com/V6TE0tn.png)
	
	
	> 记录下当前可用的镜像名称
		
		#创建虚拟机及云服务
		$vmname = 'zymvips01'
		$vipName = 'VIP1'
		$image = '0c5c79005aae478e8883bf950a861ce0__Windows-Server-2012-Essentials-20141204-zhcn'
		New-AzureVMConfig -ImageName $image -Name $vmname -InstanceSize "Small" |
		Add-AzureProvisioningConfig -Windows -AdminUsername azure -Password "Pa@!!w0rd" |
		New-AzureVM -ServiceName $serviceName -Location $location 
	
	> 请根据拓扑创建其他虚拟机，注意修改相关变量

2. 为云服务添加多个VIP
		
		#查询一下当前云服务zymvips下有几个Vip
		$deployment = Get-AzureDeployment -ServiceName $serviceName
		$deployment.VirtualIPs.Count

	> 默认每个云服务下只有一个VIP	

	![](http://i.imgur.com/FCulEQE.png)

		$vipName = 'VIP1'
		Add-AzureVirtualIP -ServiceName $serviceName -VirtualIPName $vipName
		#查询是否添加Vip
		$deployment = Get-AzureDeployment -ServiceName $serviceName
		$deployment.VirtualIPs
	
	![](http://i.imgur.com/amBvVUy.png)
	
	>1. 如果未对新添加的Vip绑定终结点,系统则不会为vip1分配一个公网ip，后续我们添加这个关联
	>2. 请按照拓扑依次添加剩余的VIP，注意之前提到的修改相关变量名称

3. 虚拟机添加终结点
		
		$endpoint = 'sslendportsonvip1'
		Get-AzureVM -ServiceName $serviceName -Name $vmname| 
		Add-AzureEndpoint -Name $endpoint -Protocol tcp -LocalPort 443 -PublicPort 443 -VirtualIPName $vipName | Update-AzureVM
	
	![](http://i.imgur.com/KSWUZ1W.png)

	>1.请按照以上语句根据拓扑依次添加VIP与终结点的关联，注意修改相关变量名称
	
	>2.这里发现一个问题：同一个云服务下不同VIP上同名的endpoint最多只能有两个，目前还不知道这个是系统设计还是系统BUG，因此建议您在给多个VIP配置endpoint的时候不要给endpoit配置一样的名字
4. 虚拟机删除终结点
		
		Get-AzureVM -ServiceName $serviceName -Name $vmname | Remove-AzureEndpoint -Name $endpoint | Update-AzureVM

5. 删除多个VIP

		Remove-AzureVirtualIP -ServiceName $serviceName -VirtualIPName $vipName -Force
	
	> 删除VIP之前要确保已经提前删除了该VIP关联过的终结点（见第5节的介绍）
6. 绑定多个VIP

		$reservedIPName = 'rvip1'
		$label = 'rvip1'
		$location = "China North"
		$serviceName = 'zymvips'
		$vipName = 'VIP1'
		New-AzureReservedIP -ServiceName $serviceName -ReservedIPName $reservedIPName -Location $location -VirtualIPName $vipName
		Get-AzureReservedIP -ReservedIPName $reservedIPName

	![](http://i.imgur.com/VBy0B9P.png)
	
7.	解除绑定VIP
		
		Remove-AzureReservedIP -ReservedIPName $reservedIPName -Force
