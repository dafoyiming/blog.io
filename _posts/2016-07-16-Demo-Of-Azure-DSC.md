# Azure Resource Group 虚拟机部署DSC（Desired State Configuration） #

----------

## DSC 简单原理 ##
![](http://i.imgur.com/AWAKYUm.png)
通过官方网站的流程图，简单介绍一下原理
1. 编写对Windows Server 服务的DSC服务配置文件
2. 将配置文件编译并发布到存储位置（可以是Azure的存储账号中或本地存储位置）同时打包相关资源文件
3. DSC抓取配置并在Azure虚拟机上执行配置文件

## 创建ARM虚拟机 ##
-  可以选择通过Ibiza Portal创建
-  可以选择通过Power Shell创建
-  可以选择通过Azure CLI创建

> 由于目前Azure China 都已经升级的资源组虚拟机。因此我们采用此类虚拟机做实验
![](http://i.imgur.com/56myG4c.png)
>  重要：建议使用Windows Server 2012 以上操作系统，并确认升级到WMF5.0(KB3134758)
>  下载链接：https://www.microsoft.com/en-us/download/confirmation.aspx?id=50395
>  使用0.8.6以上版本的Powershell，本次实验使用的是1.6.0

## 发布配置文件 ##


- 首先需要创建一个PS文件。简单理解，这个PS文件会发布到Azure存储账户或指定的存储位置。DSC会去发布位置下载这个脚本，并在开机或更新虚拟机的同时执行脚本。这里使用一个开启Windows IIS 服务的PS，如下：

IISInstall.ps :

	configuration IISInstall 
	{ 
    node (“localhost”) 
    { 
        WindowsFeature IIS 
        { 
            Ensure = “Present” 
            Name = “Web-Server”                        
        } 
    } 
	}


- 发布PS文件
	
	PS C:\Users\zhang.yiming> Publish-AzureRmVMDscConfiguration -ConfigurationPath D:\examples\IISInstall.ps1


![](http://i.imgur.com/WwtAYWx.png)

发布后可以在上图中的Blob中看到DSC配置包，也可以在Ibiza中看到

![](http://i.imgur.com/1z3NBq2.png)


## 设置虚拟机的DSC扩展 ##


> 如果虚拟机在创建过程中并未安装此扩展，则在设置DSC扩展的过程中会自动安装这个扩展。（目前经过测试通过Ibiza来安装这个扩展还有问题，不建议使用Ibiza来创建）

	PS C:\Users\zhang.yiming> Set-AzureRmVMDscExtension -ResourceGroupName zymgrp -VMName zymvm2 `
	-ArchiveBlobName IISInstall.ps1.zip -ArchiveStorageAccountName zymstore -Version '2.19' -ConfigurationName "IISInstall"

![](http://i.imgur.com/xUbOlxB.png)

> 升级的过程中大概需要10-15分钟


## 如何Troubleshooting ##

虚拟机更新完成后，我们可以看到在虚拟机上IIS服务已经被脚本自动启动

![](http://i.imgur.com/LIj4l0b.png)

如果在部署的时候发生中断，Ibiza提供了查看相关日志输出的位置，我们可以根据这些日志排查问题


![](http://i.imgur.com/3cMQEjc.png)

## 进阶测试 ##

我们可以利用DSC发布的一些Module来实现一些更复杂的部署工作，比如通过xWebAdministration实现对IIS的更进一步的配置，实现自动替换IIS默认首页及应用发布位置

- 安装DSC Resource Kit
  下载地址https://gallery.technet.microsoft.com/xAzure-PowerShell-Module-7dbf43b4
- 将安装包解压复制到C:\Program Files\WindowsPowerShell\Modules\目录下
- 测试module是否已经被加载

	PS C:\Users\zhang.yiming> Get-DscResource
	
  ![](http://i.imgur.com/wjttCiV.png)

- 创建DSC配置文件
  
	C:\examples\FourthCoffee.ps1

----------
	configuration FourthCoffee 
	{ 
		#由于可能出现多个xWebAdministration版本同时存在，因此建议添加ModuleVersion这个参数
    	Import-DscResource -ModuleName xWebAdministration -ModuleVersion 1.3.2.4 
		# Install the IIS role 
    	WindowsFeature IIS  
    	{  
        	Ensure          = “Present”  
        	Name            = “Web-Server”  
    	}  
  
    	# Install the ASP .NET 4.5 role  
    	WindowsFeature AspNet45  
    	{  
        	Ensure          = “Present”  
        	Name            = “Web-Asp-Net45”  
    	}  

    	# Stop the default website  
    	xWebsite DefaultSite 
    	{  
        	Ensure          = “Present”  
			Name            = “Default Web Site”  
        	State           = “Stopped”  
        	PhysicalPath    = “C:\inetpub\wwwroot”  
        	DependsOn       = “[WindowsFeature]IIS”  
    	}  
  
		 # Copy the website content  
		 File WebContent  
		 {  
		  	Ensure          = “Present”  
		 	SourcePath      = “C:\Program Files\WindowsPowerShell\Modules\xWebAdministration\BakeryWebsite” 
			DestinationPath = “C:\inetpub\FourthCoffee” 
		  	Recurse         = $true  
		   	Type            = “Directory”  
			DependsOn       = “[WindowsFeature]AspNet45”  
		}   
			
		# Create a new website  
		xWebsite BakeryWebSite   
		{  
			Ensure          = “Present”  
			Name            = “FourthCoffee” 
			State           = “Started”  
			PhysicalPath    = “C:\inetpub\FourthCoffee”  
			DependsOn       = “[File]WebContent”  
		} 
	}


- 将新的首页文件复制到xWebAdministration模块的目录下，DSC配置发布的时候会自动将这个目录中的首页资源一并打包的zip文件中并上传到Azure存储账号：
   
![](http://i.imgur.com/MH91ysS.png)

- 发布DSC配置文件
	
		Publish-AzureRmVMDscConfiguration -ConfigurationPath C:\examples\FourthCoffee.ps1 `
		-ResourceGroupName zymgrp -StorageAccountName zymstore

- 启动DSC配置
		
		Set-AzureRmVMDscExtension -ResourceGroupName zymgrp -VMName zymvm2 `
		-ArchiveBlobNameFourthCoffee.ps1.zip -ArchiveStorageAccountName zymstore -Version '2.19' `
		-ConfigurationName "FourthCoffee" 

   ![](http://i.imgur.com/TyE6fcO.png)

- 看一下IIS上面的配置已经生效
  ![](http://i.imgur.com/kBKsAaH.png)
  ![](http://i.imgur.com/9Kvemyk.png)