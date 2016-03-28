# 通过Azure Automation 实现虚拟机按照时间开/关机  #


- 解决之前的脚本无法通过*星期几*来设置虚拟机定时开关机的问题


##创建一个自动管理账户##
![](http://i.imgur.com/za0V8ng.png)

##创建一个RUNBOOK##
![](http://i.imgur.com/icBUMBo.png)

> 1. RUNBOOK的名称需要和脚本中的WORKFLOW名称一致

##在草稿中编写您的RUNBOOK##
![](http://i.imgur.com/aJdh4cH.png)

> 1.脚本请参考本文后面的附件，只需拷贝进去就可以

## 测试RUNBOOK ##

![](http://i.imgur.com/UCSr5Ov.png)

![](http://i.imgur.com/EcgLrme.png)

![](http://i.imgur.com/K6ZZkkV.png)

> 1. 请按照参数说明填写
> 2. DAYSOFWEEKTOEXECUTE
     -：表示不备份 X：表示备份
     -XXXXXX- 表示周日和周六不备份

## 发布RUNBOOK ##

![](http://i.imgur.com/h8pI4Lg.png)

## 设置计划日程 ##

![](http://i.imgur.com/gaXgH6g.png)

![](http://i.imgur.com/mMFdJQv.png)

> 1.可以指定计划每天执行的时间。根据本文的例子如果在周末执行RUNBOOK则不会执行关机操作。但会输出日志"Skipping this day of the week"

![](http://i.imgur.com/KwhZqZp.png)

> 1.请重新填写一遍之前的参数

## RUNBOOK 内容 ##

    Workflow stop-azurevms
    {
    Param
    (   
        [Parameter(Mandatory=$true)]
        [String]
        $subscriptionName,

        [Parameter(Mandatory=$true)]
        [String]
        $username,
        
        [Parameter(Mandatory=$true)]
        [String]
        $password,
        
        [Parameter(Mandatory=$true)]
        [String]
        $vmName,

        [Parameter(Mandatory=$true)]
        [String]
        $cloudServiceName,

        [Parameter(Mandatory=$true)]
        [String]
        $daysOfWeekToExecute
    )

    $dayOfWeek = [int](Get-Date).DayOfWeek + 1
    Write-Output "DayOfWeek $dayOfWeek"

    $dayToExecute = ""
    if ($daysOfWeekToExecute.Length -gt $dayOfWeek)
    {
        $dayToExecute = $daysOfWeekToExecute.substring($dayOfWeek - 1, 1)
    }
    Write-Output "DayToExecute value: $dayToExecute"
    if ($dayToExecute -eq "X" -or $dayToExecute -eq "x")
    {
        
        $pwd = ConvertTo-SecureString -String $password -AsPlainText -Force
        $credentials = New-Object System.Management.Automation.PSCredential($username, $pwd)
        Write-Output $credentials

        Add-AzureAccount -Credential $credentials -Environment AzureChinaCloud

        Write-Output "Select subscription"
        Select-AzureSubscription -SubscriptionName $subscriptionName

        Write-Output "Stopping VM"
        Stop-AzureVM -ServiceName $cloudServiceName -Name $vmName -StayProvisioned
        Write-Output "VM has been stopped"
    }
    else
    {
        Write-Output "Skipping this day of the week"
    }
	}

> 1.虚拟机开机RUNBOOK可以通过修改 
> Stop-AzureVM -ServiceName $cloudServiceName -Name $vmName -StayProvisioned 
> 为Start-AzureVM -ServiceName $cloudServiceName -Name $vmName命令来实现