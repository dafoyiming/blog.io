# 利用Azure Automation功能实现自动发送Office 365邮件 #


----------
> 本文使用Office 365提供的smtp服务发送邮件，使用本人的Office 365账户为例。

## 添加认证资源 ##

> Runbook要使用到Office 365的登录认证，因此需要在“资产”中添加Office 365登录信息

![](http://i.imgur.com/afnFa9Y.png)

![](http://i.imgur.com/jzHU97N.png)

## 运行Runbook时需要填写的参数 ##

![](http://i.imgur.com/SmoXO1F.png)

![](http://i.imgur.com/7NXVoiS.png)

> 发件箱需要填写您Office 365的账号
> 其他字段请根据您的需求填写

## Runbook的修改 ##

runbook中需要填写中国的Office 365的smtp地址

    -UseSsl
    -Port 587 
    -SmtpServer 'smtp.partner.outlook.cn'
    -From $From 

## 以下为Runbook ##

    workflow SendMailUsingOffice365
	{
    
      Param
    (            
       # [parameter(Mandatory=$true)]
       # [String]
       # $Subject,

        # PowerShell Credentials for the Secure SMTP Service
	[parameter(Mandatory=$true)]
        [String]
        $AzureOrgIdCredential = "enSence Office 365", 
        
        [parameter(Mandatory=$true)]
        [String]
        $Body = "This is an automated mail send from Azure Automation relaying mail using Office 365.",
                        
        [parameter(Mandatory=$true)]
        [String]
        $To ='SomeEmail@domain.com',
        
        [parameter(Mandatory=$false)]
        [String]
        $Cc ='SomeEmail@domain.com',
        
        [parameter(Mandatory=$false)]
        [String]
        $Bcc ='SomeEmail@domain.com',
               
        [parameter(Mandatory=$true)]
        [String]
        $From ='SomeEmail@domain.com' 
    )
    

 
     #$AzureOrgIdCredential = "enSence Office 365"
     #$Body = "This is an automated mail send from Azure Automation relaying mail using Office 365."
     $Subject = "Mail send from Azure Automation using Office 365"
     #from='psd@ensence.com'
 
     
    # Get the PowerShell credential and prints its properties
    $Cred = Get-AutomationPSCredential -Name $AzureOrgIdCredential
    if ($Cred -eq $null)
    {
        Write-Output "Credential entered: $AzureOrgIdCredential does not exist in the automation service. Please create one `n"   
    }
    else
    {
        $CredUsername = $Cred.UserName
        $CredPassword = $Cred.GetNetworkCredential().Password
        
        Write-Output "-------------------------------------------------------------------------"
        Write-Output "Credential Properties: "
        Write-Output "Username: $CredUsername"
        Write-Output "Password: *************** `n"
        Write-Output "-------------------------------------------------------------------------"
       # Write-Output "Password: $CredPassword `n"
    }

     
     Send-MailMessage `
    -To $To  `
    -Cc $Cc  `
    -Bcc $Bcc `
    -Subject $Subject  `
    -Body $Body `
    -UseSsl `
    -Port 587 `
    -SmtpServer 'smtp.office365.com' `
    -From $From `
    -BodyAsHtml `
    -Credential $Cred
  
        Write-Output "Mail is now send `n"
        Write-Output "-------------------------------------------------------------------------"

	}


> 由于排版问题以上Runbook省略了备注的内容