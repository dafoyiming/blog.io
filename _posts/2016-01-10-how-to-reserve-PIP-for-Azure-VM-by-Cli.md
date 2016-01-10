# 如何通过Azure Cli 创建PIP #

----------


## 下载Azure PublishsettingsFile ##
	
	`C:\Users\zhang.yiming>azure account download -e AzureChinaCloud`

![](http://i.imgur.com/ai18X7X.png)



> 请记录文件下载后的保存路径，导入时会用到
## 导入Azure PublishsettingsFile ##

    `C:\Users\zhang.yiming>azure account import c:/Internal-004-Internal-002-WATSTest03
	 -Internal-003-Internal-005-Internal-001-12-27-2015-credentials.publishsettings`

## 设置默认管理订阅 ##
  
- 列出目前可用的订阅
  
	`C:\Users\zhang.yiming>azure account list`
	![](http://i.imgur.com/0UkkjXJ.png)
  
- 将需要设置PIP的虚拟机的订阅设置为默认订阅

	`C:\Users\zhang.yiming>azure account set 4c1f7e7c-****-****-****-****b58636`
  	 ![](http://i.imgur.com/I8zF17J.png)

	> 安全原因部分信息被隐去，请按照实际情况填写

- 查看虚拟机是否设置过PIP
    
	`C:\Users\zhang.yiming>azure vm public-ip list zymub`

	![](http://i.imgur.com/LxObm1m.png)

- 为虚拟机设置PIP

	`C:\Users\zhang.yiming>azure vm public-ip set zymub zymubpip`

	![](http://i.imgur.com/mht9iWK.png)

- 可以在检查一下当前的PIP
	
	`C:\Users\zhang.yiming>azure vm public-ip list zymub`
	
	![](http://i.imgur.com/TBxOHCy.png)
	
	`C:\Users\zhang.yiming>azure vm show zymub`

	![](http://i.imgur.com/AP62cEh.png)