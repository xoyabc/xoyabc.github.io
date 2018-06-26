---
layout: post
title: 使用powershell删除指定日期前的文件并添加到计划任务
categories: windows
description: 使用powershell删除指定日期前的文件并添加到计划任务
keywords: windows, powershell
---

  公司会议视频转换所使用的机器系统为windows，因程序转换失败时会在D盘生成临时文件，日积月累会造成磁盘满，造成新的视频会议录制
内容无法转换，为避免此类情况，临时用powershell编写了对应的删除及添加计划任务脚本。

系统环境：`Microsoft Windows Server 2012 R2 Standard`

powershell版本：4.0

## 1. 删除脚本

这里放到了D盘根目录下面，文件名为`D:\delete_tmp_files.ps1`，删除15天前的文件和目录。

删除脚本`delete_tmp_files.ps1`内容如下：

``` powershell
#delete old tmp files,just save files in 15 days~
$TimeOutDays=15    
$filePath="D:\tmp"     
$allFiles=get-childitem -path $filePath     
foreach ($files in $allFiles)     
{       
   $daypan=((get-date)-$files.lastwritetime).days       
   if ($daypan -gt $TimeOutDays)       
   {         
     remove-item $files.fullname -Recurse -force       
    }     
}
```
`-Recurse`表示递归，可以删除子目录

`-force` 强制删除，可以删除隐藏及只读文件(delete all file force fully，delete all hidden or read-only files)



## 2. 添加计划任务

选中`add_schedule_task.ps1`脚本，鼠标右键选择`使用PowerShell运行`即可。

加入计划任务的powershell脚本`add_schedule_task.ps1`内容如下：

``` powershell
ipmo PSScheduledJob 
$T = New-JobTrigger -Weekly -DaysOfWeek 0,1,2,3,4,5,6 -At 2:38AM
Register-ScheduledJob -Name Delete-Tmp-Files -FilePath "D:\delete_tmp_files.ps1" -Trigger $T
```

`-DaysOfWeek`: 在周计划任务中，指定每周的哪一天运行，一般与`-Weekly`配合使用。

周日-->周一-->...-->周六可用对应英文表示，也可用数字表示，对应表如下：

| 表示方法 | 周日 | 周一 | 周二 | 周三 | 周四 | 周五 | 周六 |
| :------:| :--: |:--: |:--: |:--: |:--: |:--: |:--: |
| 英文 | Sunday | Monday | Tuesday | Wednesday | Thursday | Friday | Saturday
| 数字 | 0 | 1 | 2 | 3 | 4 | 5 | 6 |

## 3.查看计划任务

点击`服务器管理器`-->`任务计划程序`-->`Microsoft`-->`Windows`-->`PowerShell`-->`ScheduledJobs`

![1.jpg](https://i.loli.net/2018/06/27/5b3275ba4f779.jpg)
![2.jpg](https://i.loli.net/2018/06/27/5b3275c47f58c.jpg)

参考：

[利用powershell删除早于某个指定日期的文件](http://blog.51cto.com/281816327/1436751)

[delete-files-older-than-15-days-using-powershell](https://stackoverflow.com/questions/17829785/delete-files-older-than-15-days-using-powershell)

[HOW TO CREATE SCHEDULE TASK USING POWERSHELL](https://gallery.technet.microsoft.com/scriptcenter/for-windows-7-and-less-1042e194)

[New-JobTrigger](https://docs.microsoft.com/en-us/powershell/module/psscheduledjob/new-jobtrigger?view=powershell-5.1)

[how-to-delete-a-folder-or-file-using-powershell](http://dotnet-helpers.com/powershell-demo/how-to-delete-a-folder-or-file-using-powershell/)


