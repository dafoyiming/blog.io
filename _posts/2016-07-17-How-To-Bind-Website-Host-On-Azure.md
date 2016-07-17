# 部署于Azure VM上的Web站点如何防止恶意域名绑定 #

----------
## 背景说明 ##

有这样的场景，客户部署于Azure VM上的Web站点的合法域名，如zymvm2.chinaease.cloudapp.chinacloudapi.cn被一个非法的域名绑定了Cname。进而在互联网进行ICP认证检查的时候被Azure ICP技术团队发现并将用户的Azure订阅错误封停。

## 解决方案 ##

### - IIS 进行site Bindings ###

首先，我们将Azure IIS站点zymvm2.chinaease.cloudapp.chinacloudapi.cn 做Cname 映射到 azure.dafoyiming.cn

![](http://i.imgur.com/kicgrzm.png)

----------
![](http://i.imgur.com/Yzmo9iO.png)

----------
![](http://i.imgur.com/tzFkM34.png)

> 我们这里假设azure.dafoyiming.cn这个域名就是那个未经过ICP认证的恶意域名；www.zymvm2.com这个域名为客户合法的域名

修改IIS site Bindings

![](http://i.imgur.com/pgYE44y.png)

修改Host Name 为 www.zymvm2.com后，我们再次访问azure.dafoyiming.cn 发现这样的恶意绑定已经无法生效，同时只有合法的域名才能被绑定

![](http://i.imgur.com/7VXktOv.png)

### - Apache 进行VirtualHost 重定向 ###

首先，我们将Apache 站点zymvm1.chinaeast.cloudapp.chinacloudapi.cn 做Cname 映射到 azure.dafoyiming.cn
![](http://i.imgur.com/xybO7H8.png)

----------
![](http://i.imgur.com/FYNqwEe.png)

----------

![](http://i.imgur.com/CvsGSQa.png)

> 我们这里假设azure.dafoyiming.cn这个域名就是那个未经过ICP认证的恶意域名；zym.dafoyiming.cn这个域名为客户合法的域名

修改Apache 配置文件


> 假设当前站点的配置文件为00-default.conf,同时我们需要在/var/www/errorpage目录下部署一个报错文件

	[root@zymvm1 conf.d]# vim /etc/httpd/conf.d/00-default.conf

----------

	NameVirtualHost *
	<VirtualHost *:80>
        DocumentRoot /var/www/errorpage
	</VirtualHost>
	<VirtualHost *:80>
        DocumentRoot /var/www/html
        ServerName zym.dafoyiming.cn
        ServerAlias zymvm1.chinaeast.cloudapp.chinacloudapi.cn
	</VirtualHost>

----------
	[root@zymvm1 conf.d]# vim /var/www/errorpage/index.html

----------
    error Site.

当我们再次访问http://azure.dafoyiming.cn 的时候
![](http://i.imgur.com/803zCLh.png)

现在我们将域名zym.dafoyiming.cn绑定Cname到zymvm1.chinaeast.cloudapp.chinacloudapi.cn后，我们发现可以正常访问
![](http://i.imgur.com/6F00Ufy.png)

----------
![](http://i.imgur.com/BvTcDAv.png)


----------
至此，我们可以通过以上方法分别在IIS和Apache上实现防止恶意域名的绑定