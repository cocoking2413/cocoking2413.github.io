 
##powershell记录  

####record code

        1.获取mac
                getmac /fo csv | convertfrom-csv
        2.下面是一个快速获取分配到你电脑的所有IP地址的方法:
                $ipaddress = [System.Net.DNS]::GetHostByName($null)
                Foreach ($ip in $ipaddress.AddressList)
                {
                $ip.IPAddressToString
                }
                假如你用一个主机名 (例如：”server123″)替换$null, 你可以获取分配到指定主机的IP地址。
                如果你只对IPv4地址感兴趣的话，可以尝试下面的代码：

                $ipaddress = [System.Net.DNS]::GetHostByName($null)
                foreach($ip in $ipaddress.AddressList)
                {
                if ($ip.AddressFamily -eq 'InterNetwork')
                {
                $ip.IPAddressToString
                }
                }

#####Powershell常用命令
        1.Get-Command 得到Powshell所有命令
        2.Get-Process 获取所有进程
        3.Set-Alias  给指定命令重命名 如:Set-Alias aaa Get-Command 
        4.Set-ExecutionPolicy remotesigned 设置powershell可直接执行脚本文件 一般脚本文件以.ps1结尾 执行脚本文件直接输入文件地址即可执行  脚本文件中只写命令即可
        5.Get-Help get-* 查询以get开头的命令   Get-Help *service*  Get-Help Get-Command 获取Get-Command命令的基本用法
        6.Get-Member 获取对象属性  如: $var | Get-Memeber  访问$var属性 直接$var.ToString()


#####PS中的变量定义
        不需要定义或声明数据类型
        在变量前加"$"
        定义变量的规则
        -变量可以是数字 $123
        -变量可以是字符串 $abc
        -变量可以是特殊字符 ${@1b}
        内置的变量
        -$pshome
        -$home
        -$profile

        变量赋值: $var=123  $var="aaaaaa"
        取变量值: $var

        变量赋值方法:Set-Variable var 100
        取值方法:    Get-Variable var
        清空值:      Clear-Variable var
        删除变量     Remove-Variable var
        取多个变量如var1 var2 var3地值:   Get-Variable var*

        另一种赋值方法 $var1="bbb"    $var2="$var $var1"  结果$var2="aaaaaa bbb"
                    $var2='$var $var1' 结果$var2="$var $var1"
        $date=Get-Date  获取当前时间
        $date.AddDays(3) 当前时间加3天

 

#####排序用法
        Get-Process | Sort-Object ws   根据WS值由小到大排序
        Get-Process | Sort-Object | fl    Get-Process | Sort-Object | Format-List  以列表形式显示数据

#####导入导出文件
        Get-Process > c:/aa.txt
        Get-Process | Export-Clixml c:/ddd.xml  将命令执行结果导出到xml文件中
        Import-Clixml c:/ddd.xml  将xml文件导出到控件台


#####注释使用
        Get-Proccess | #这里写注释信息
        sort ws

######比较运算符
        $var="abc"
        $var -like "&b&"  返回true
        $var -clike "&b&"   返回true

 

######函数使用
        案例:在一个脚本文件中有如下代码:
        $var1=10
        function one{"The Variable is $var1"}
        function two{$var1=20;one}
        one 
        two
        one
        执行结果: The Variable is 10
            The Variable is 20
                The Variable is 10
        此示例表明,在函数中改变变量值并不影响实际值
        若需改变其值请看如下代码:
        $var1=10
        function one{"The Variable is $var1"}
        function two{$Script:var1=20;one}
        one 
        two
        one
        执行结果: The Variable is 10
            The Variable is 20
                The Variable is 20

 

#####foreach使用
        $var=1..6  #定义数组
        foreach($i in $var)
        {
        $n++
        Write-Host "$i"
        }
        Write-Host "there were $n record"


#####if使用
        Get-Service | foreach{
        if($_.status -eq "running"){
            Write-Host $_.displayname  "("$_status")" -foregroundcolor "green"
        }
        else
        {
            Write-Host $_.displayname  "("$_status")" -foregroundcolor "red"
        }
        }

 

#####error使用
        function one
        {
        Get-Process -ea stop
        Get-ChildItem ada -ErrorAction stop  #此句有误
        Get-Process -ErrorAction stop
        }
        one

        -ea 定义当错误发生以后该如何继续执行
        $?可以测试命令执行成功还是失败,成功则结果为true 反之为false

 

#####单步调试
        先设置Set-PSDebug -step
        for($i=1;$i -le 10;$i++)
        {
        Write-Host "loop number $i"
        }