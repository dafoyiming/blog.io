# 如何创建Azure ILB并实现多端口负载及验证 #

----------

## 系统原理图 ##
> 对Azure官方提供的原理图略作标注
![](http://i.imgur.com/OQRFpxE.png)

## 设置ILB ##
    $svc="zymilb"
	$ilb="zym"
	$subnet="subnet01"
	$IP="10.0.0.10"
	Add-AzureInternalLoadBalancer -ServiceName $svc -InternalLoadBalancerName $ilb –SubnetName $subnet –StaticVNetIPAddress $IP

## 添加虚机到ILB ##

### 添加UDP端口 ###
	$svc="zymilb"
	$vmname="zym01"
	$epname="InternalUDP"
	$lbsetname="UDP"
	$prot="udp"
	$locport=53
	$pubport=53
	$ilb="zym"
	Get-AzureVM –ServiceName $svc –Name $vmname | Add-AzureEndpoint -Name $epname -LbsetName $lbsetname -Protocol $prot
	-LocalPort $locport -PublicPort $pubport –DefaultProbe -InternalLoadBalancerName $ilb | Update-AzureVM

### 添加TCP端口 ###
	$svc="zymilb"
	$vmname="zym01"
	$epname="InternalTCP"
	$lbsetname="TCP"
	$prot="tcp"
	$locport=80
	$pubport=80
	$ilb="zym"
	Get-AzureVM –ServiceName $svc –Name $vmname | Add-AzureEndpoint -Name $epname -LbsetName $lbsetname -Protocol $prot
	-LocalPort $locport -PublicPort $pubport –DefaultProbe -InternalLoadBalancerName $ilb | Update-AzureVM`

> zym02、zym03 只需修改$vmname变量即可

## 验证端口 ##

### 测试 UDP端口（53）###
- 分别在zym01，zym02，zym03上抓包
	
	`[azureuser@zym01 ~]$ sudo tcpdump -i any -vnn udp port 53`

- 在zymclient上多次向ILB VIP 扫描53端口
	
	`[azureuser@zymclient ~]$ nc -vuz 10.0.0.10 53`

- 在zym01上发现这样的UDP包
	
	**12：35:47.543602** IP (tos 0x0, ttl 64, id 2501, offset 0, flags [DF], proto UDP (17), length 29) **10.0.0.9.41723** > 10.0.0.6.53: [|domain]

- 在zym02上发现这样的UDP包

	**12:37:28.548466** IP (tos 0x0, ttl 64, id 37980, offset 0, flags [DF], proto UDP (17), length 29) **10.0.0.9.37642 **> 10.0.0.7.53: [|domain]

- 在zym03上发现这样的UDP包
	
	**12:36:44.057337** IP (tos 0x0, ttl 64, id 58980, offset 0, flags [DF], proto UDP (17), length 29) **10.0.0.9**.59616 > 10.0.0.8.53: [|domain]

> 说明在这个时间段内三台机器分别收到了来自zymclient并由ILB转发过来的53端口请求

### 测试TCP端口（80）###
- 分别在zym01，zym02，zym03上抓包

	`[azureuser@zym01 ~]$ sudo tcpdump -i any -vnn tcp port 80`

- 在zymclient上多次请求80端口服务

	`[azureuser@zymclient ~]$ curl 10.0.0.6:80`

- 在zym01上发现这样的TCP包

	**13：09:20**.088717 IP (tos 0x0, ttl 64, id 7110, offset 0, flags [DF], proto TCP (6), length 2864) **10.0.0.6.80 **> **10.0.0.9**.56679: Flags [.],

- 在zym02上发现这样的TCP包
	
	**13:09:35**.086617 IP (tos 0x0, ttl 64, id 7110, offset 0, flags [DF], proto TCP (6), length 2864) **10.0.0.6.80 **> **10.0.0.9**.58823: Flags [.]

- 在zym03上发现这样的UDP包

	**13:09:56**.082415 IP (tos 0x0, ttl 64, id 7110, offset 0, flags [DF], proto TCP (6), length 2864) **10.0.0.6.80 **> **10.0.0.9**.56548: Flags [.]

> 说明在这个时间段内三台机器分别收到了来自zymclient并由ILB转发过来的80端口的请求

## 如何在Centos系统上安装Http服务 ##
- 安装Httpd包
	
	sudo yum install -y httpd

- 启动Http服务

	sudo service httpd restart

## 一个重要的问题 ##

当客户自己添加一个自定义的UDP端口的时候，发现ILB并没有做任何转发！！

### 原因 ###

ILB和SLB的原理其实是一样的都是通过Probe Protocol去侦测端口是不是up。目前azure支持的Probe protocol 有两种“tcp”，“http”。因此，基于UDP的端口（以9009为例）无法被ILB监测到，所以被认为端口是Down的，因此就不转发了

![](http://i.imgur.com/HG8Oipa.png)

### 关于 UDP 53 端口 ###

此时，你肯定要问那为什么53 端口可以！！ 我们来看看53端口在服务器上的状态你就明白了

![](http://i.imgur.com/SlevF3f.png)

原来如此，在azure上53端口同时开了两种协议TCP和UDP。ILB能够通过TCP监测到53端口是up的。所以53端口可以访问，因此ILB认为会转发请求

### 解决方案 ###

针对UDP端口不做Probe监测。在创建添加endpoint的时候我们可以指定-Noprobe，这样问题就解决了

	Get-AzureVM –ServiceName $svc –Name $vmname | Add-AzureEndpoint -Name $epname -LbsetName $lbsetname -Protocol $prot
	-LocalPort $locport -PublicPort $pubport –NoProbe -InternalLoadBalancerName $ilb | Update-AzureVM`