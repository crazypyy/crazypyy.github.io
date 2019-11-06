---
title: powershell 处理敏感文件
description: 给公司写一个扫描敏感文件的程序
categories:
 - 公司
tags:
 - 公司
 - powershell
---

# 前言
最近公司需要对一些敏感文件进行扫描，正好最近在学习powershell的黑客手法，准备用powershell写这个程序(我真想把当时的自己打一顿，python不香了吗)
当然本文所涉及的规则和一些内容是脱敏的哈，大致的功能代码会给出来的。


# 使用Powershell 扫描敏感文件
##  编辑器
用的系统自带的ISE，当然在写代码前得做几步操作
` Get-ExecutionPolicy`    获取当前的安全策略，在没有改动的情况下 应该是 Restricted	
` Set-ExecutionPolicy`    设置当前的安全策略 为 bypass 就可以外部引用ps1脚本。

至于具体的powershell教程，可以看我博客里的另一篇文章

## 功能
主要功能分为三个部分，
1. 全盘文件扫描
2. 全盘文件类型匹配与正则匹配
3. 上传文件到指定的ftp服务器

### 全盘文件扫描
其实这一块很简单，就一句话
`Get-ChildItem -resure`  这句就是直接遍历整个文件夹下的所有内容，只是我们这边的输出不使用管道符，因为在实际操作中，速度是一个比较重要的因素，使用管道符重定向输出的话，程序就跑的很慢。所以使用底层的io输出 `[System.IO.File]::WriteAllLines()` 这句话几乎可以使程序快十倍。

理清全盘文件扫描的主要函数功能后，主要的工作就是细化了，比如获取所有盘符，检查错误之类的。

在这里，由于powershell 遇到一些报错后，可以继续执行下去，所以我就没有try catch。

这一部分的全部代码如下

```powershell
####  文件检索与输入到特定文件中
####  D:\xls1.txt D:\xls2.txt D:\xls3.txt D:\xls4.txt
function file_search(){
    Write-Host -ForegroundColor red -Object "全盘文件检索开始"
    foreach($disk_single in $disk){
        $content = Get-ChildItem $disk_single -Recurse -Filter *.xlsx
        
        $absolute = @()
        foreach($a in $content){
            $absolute += $a.fullname
    
        }

        $absolute.count
        $a = ($absolute.count-$a.Count%4)/4
        Write-Host -ForegroundColor Blue -Object "正在写$disk_single$file1"
        [System.IO.File]::WriteAllLines($disk_single+$file1,$absolute[0..($a-1)],[text.encoding]::Unicode)
        Write-Host -ForegroundColor Blue -Object "正在写$disk_single$file2"
        [System.IO.File]::WriteAllLines($disk_single+$file2,$absolute[$a..($a*2-1)],[text.encoding]::Unicode)
        Write-Host -ForegroundColor Blue -Object "正在写$disk_single$file3"
        [System.IO.File]::WriteAllLines($disk_single+$file3,$absolute[($a*2)..($a*3-1)],[text.encoding]::Unicode)
        Write-Host -ForegroundColor Blue -Object "正在写$disk_single$file4"
        [System.IO.File]::WriteAllLines($disk_single+$file4,$absolute[($a*3)..($absolute.count-1)],[text.encoding]::Unicode)
        $content.count
    }
    Write-Host -ForegroundColor Red -Object "文件检索程序结束"

    
}
```
当初是准备分成四份文件，然后起多进程的，后来发现，其实不用，哈哈哈，就不管了。至于内容，我贴的这个是当时测试用的处理excel的，也就是匹配excel文件。

### 正则匹配
这里涉及到核心匹配规则，我就瞎编一套给大家看看，反正公司内部用的不是这么回事。
就用身份证匹配来查查，首先根据已经查询到的excel，读取其内容，然后一个一个匹配就可以了，当时说起来思路是很简单的，然而实际上跑的时候很揪心。
主要函数功能是读取excel，这里使用是com接口。具体的函数作用可以查看官网上的说明。
```powershell
$excel = New-Object -ComObject excel.application
$content = $excel.Workbooks.Open($file_single,$false,$true)
$item = $content.Worksheets.Item($ws.Name)
$value = $item.Cells.Columns.Item($i).value2
```
在这里，我不得不吐槽，在读取以列数据的时候，明明一维数组就可以解决的，我提取了半天愣是报错，最后查看数据类型，才发现是二维数组，关键是什么，关键是这组数据只有一列啊，一维数组不久解决了。

这一部分的整体代码如下，当然是当时测试用的代码
```powershell
function file_patter(){
    $file_target_excel = @()
    $file_target_excel += [System.Net.Dns]::GetHostAddresses('')
    $file_target_excel_set = ''
    
    $patter = '^[1-9]\d{5}(18|19|([23]\d))\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]'
    $code = 0
   
    $file_all = $file1,$file2,$file3,$file4
    foreach($disk_single in $disk){
        foreach($file_s in $file_all){
            $file_path = "$disk_single$file_s"
            $file = Get-Content $file_path
            Write-Host -ForegroundColor Red -Object $file_path==="该文件进行正则匹配"
            foreach($file_single in $file){
                $excel = New-Object -ComObject excel.application
                $excel.Application.AskToUpdateLinks = $false
                $excel.Application.DisplayAlerts =$false
                $excel.Application.DisplayClipboardWindow =$false
                #$excel.Application.DisplayCommentIndicator =$false
                #$excel.Application.DisplayDocumentActionTaskPane=$false
                $excel.Application.DisplayDocumentInformationPanel=$false
                $excel.Application.DisplayFormulaAutoComplete=$false
                Write-Host -ForegroundColor Green "正在处理"$file_single
                try{
                    $content = $excel.Workbooks.Open($file_single,$false,$true)
                    $sheet = $content.WorkSheets
                    foreach($ws in $sheet){
                        $item = $content.Worksheets.Item($ws.Name)
                        for($i=1;$i -lt 100;$i++){
                            $value = $item.Cells.Columns.Item($i).value2
                            if($value[1,1] -eq $null){
                            break
                            }
                        for($j=1;$j -lt 100;$j++){
                            if($value[$j,1] -match $patter){
                                Write-Host -ForegroundColor red $file_single====="文件身份证匹配成功"
                                $file_target_excel += $file_single
                                Write-Host -ForegroundColor Red -Object $file_target_excel
                                $code = 1
                                break
                            }
                        }
                    if($code -eq 1){
                        break
                    }  
                }
                if($code -eq 1){
                    $code = 0
                    break
                }
            }
                $content.Close()
                $excel.Quit()
                  }
            catch{
                Write-Host -ForegroundColor DarkYellow $Error[0]
                $content.Close()
                $excel.Quit()
            }
            }
        Write-Host -ForegroundColor Red -Object $file_path==="文件内容匹配完毕"
    }
    }
    Write-Host -ForegroundColor Red -Object "全局文件检索完毕，开始写入"
    [System.IO.file]::WriteAllLines($file_target_excel_set,$file_target_excel,[text.encoding]::UTF8)
    Write-Host -ForegroundColor Red -Object "写入完毕"
}
```
### 结果上传
最后的功能就是结果上传，这一部分将最后扫描到的敏感文件上传到指定的ftp服务器上。
关键的几行代码如下：
```powershell
function Upload_File(){
    $random =   1..1000 | Get-Random
    $WebClient = New-Object System.Net.WebClient
    $WebClient.Credentials = New-Object System.Net.NetworkCredential()
    $WebClient.UploadFile()
}
```

## 结语
中间在测试的时候，顺顺利利，当到了实际使用，我了个去，什么鬼情况都有。
1. 针对excel关闭时弹窗的问题，提前设置某些属性为false就可以阻止弹窗，如果不设置，就需要手动关闭弹窗程序才能继续执行下去。
2. 需要持续关闭excel，不然内存马上就爆了。
3. 少用管道符，多用系统底层的处理，能快很多。