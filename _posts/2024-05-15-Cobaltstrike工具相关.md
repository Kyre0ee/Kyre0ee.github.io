---
layout: post
title: CobaltStrike相关内容
date: 2024-05-17 09:30
tags: C2
excerpt: "本文主要分析cobalt-strike如何进行C2链接。"
toc: true
---
# 隧道工具Cobalt-strike
Cobalt-strike接受客户端连接、协调 Beacon 植入的远程命令、提供 UI 管理和各种其他功能。
1.主动探测
2.网络指纹技术
3.被动流量检测方法
HTTP事务的编码和加密技术。
Cobalt Strike的Beacon命令与控制(C2)通信所使用的先进、灵活的流量配置文件，以逃避防御者的检测。
Beacon与Team server进行通信。它们之间的C2通信流量可以使攻击者能够轻松有效的将恶意流量伪装成正常的良性流量。（通过Malleable C2 的模块化、可扩展的特定领域语言实现）。
可以使用预制或自定义的Malleable C2配置文件。

1.Team Server收到特制HTTP请求时的行为方式。
2.可以推断出什么样的网络指纹以及有多大的置信度。
3.Beacon和Team Server之间一些真实恶意C2的详细信息。
## 主动探测与指纹识别技术
### 1. 通过HTTP HTTP/s选项请求和响应指纹
* 当服务端收到请求通过HTTP options方法，服务端会回复HTTP status code 200和Content-Length: 0
  ![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/1a7e9eae-2644-498c-8410-842f802bbfe3)
2.HTTP/HTTPS GET 请求和响应指纹
* 请求stager URI,当用户向Team server发送以下HTTP请求时，服务器向客户端返回32位Beacon二进制。（如果URL以/开头，HTTP服务器将返回404）。如果配置文件将host_stage设置为false则URL stager和stager64将被屏蔽。
  ![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/836c71a3-011d-488d-be2b-a35c572a9ac2)
  ![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/d7d32a68-b873-4796-8ac8-a325a8c54cd3)
* 请求Beacon.http-get URI，可以在Malleable C2配置文件中配置某些预设URL路径以提供静态数据。如果用户向URL beacon.http-get发送GET请求，Team Server将使用其配置文件中指定的数据进行响应（发送http-get配置的服务器标记内的输出部分）。如果输出部分仅包含命令print;则服务器会使用HTTP状态吗200和Content-Length：0进行响应。
  ![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/a6d7b4d0-f2ae-479a-9d34-318bbe58bd89)
![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/a2a663d6-36a7-4aa8-b76c-94282c7c9f90)
* 请求Beacon.http-post URI,同上述配置
  ![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/41c0e28b-7b83-4e18-bfb7-c5072aee9f02)
![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/f0e872d5-7f26-4865-a3fd-7451ac3ffe6c)
* URI 校验和
Team Server使用请求URI的自定义1字节校验和作为提供Beacon二进制文件的条件。
![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/e9f9ac76-0720-4b4e-8495-2bb4725be18d)
对于32位payload，代码将URL校验和结果与文字整数92L进行比较。对于64位payload，算法将校验和与93L进行比较。如果校验满足条件之一，服务器将使用适当的Beacon二进制文件的原始字节进行响应。
![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/d627128e-c7c2-4544-b809-f0712d9caa4b)
* 随机URI
  当用户发送随机URI路径，Team Server将使用Content-Length：0的404响应
  ![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/df7b254f-ba90-4773-ab2f-530b0cbed1b4)
### 2.通过DNS主动探测
Cobalt Strike 的DNS侦听器使Beacon能够利用DNS协议与Team server进行通信。
基于DNS的Beacon使用DNS TXT、AAAA和A记录来实现任务监控和其他功能。
* 使用TXT记录，收到请求后Team server会在TXT记录响应中使用base64编码的Beacon二进制文件进行响应。
  ![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/53e7decf-33d7-4827-9175-14cb59f88f0b)
![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/ffaa9cb6-1309-4187-9ea5-f459feab6e9c)
主动探测过程：
1.利用Shodan的源服务收集潜在的Team Server的IP地址
2.发送32位stager探测
3.候选者返回预期的Cobalt Strike响应
4.初始化与netcat的TCP连接，以测试、验证和提取所提供的stager字节
![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/58545216-82f9-4408-bb15-42866ae21cc4)
![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/d07e5f38-634b-484f-aa78-0a94cb238877)

## C2数据加解密
Beacon会使用RSA算法公钥对元数据（与受害者系统有关信息）进行加密，并将其发送到Cobalt Strike TeamServer。
TeamServer将使用私钥恢复Beacon明文元数据以区分Beacon客户端。此外，可以从解密的元数据中提取AES对此密钥。
Beacon和TeamServer可以使用AES密钥对进一步的请求和响应数据进行加密和解密，以完成C2流量数据。
### 数据加解密算法
Beacon与TeamServer通信使用对称（AES）和非对称（RSA）加密密钥算法组合。TeamServer会创建一个新的公钥/私钥组合并将密钥对存储在.cobaltstrike.beacon_keys 文件中。
非对称密钥算法使用RSA/ECB/PKCS1Padding
对称密钥算法使用AES/CBC/NoPadding格式来加密解密数据
AES算法使用硬编码初始化向量（IV）进行初始化，静态IV为abcdefghijklmnop
![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/27da87a1-eab2-416e-8afd-4050487aeca2)

### 元数据的架构定义
元数据包含受害者有关信息。不同版本的Cobalt Strike解密后的数据包含不同的信息，
![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/4db622fd-eea2-4f52-b010-ca6088663e33)
元数据遵循结构化格式，
开头magic(0xBEEF)为4字节
数据字段的大小为4字节
Beacon生成的随机字节为16字节，对于每个进程运行都是唯一的。Beacon和TeamServer使用这些字节来创建AES密钥和HMAC密钥。该过程计算字节的SHA256哈希值。SHA256的前16字节被指定为对称加密的AES密钥，其余16字节是消息认证码的HMAC密钥。
ANSI字符集2个字节。
OEM字符集2个字节。
BeaconID四个字节。
受害者机器上Beacon的进程ID四个字节。
端口2个字节。
flag1字节。
Beacon版本号2字节。这些字节被转换为带有.的字符串。插入它们之间，例如A.B
Beacon的构建版本2字节。
为函数指针添加前缀4字节，只有架构64位才用，32位这4字节被丢弃。
指向GetModuleHandleA指针4字节
指向GetProcAddress指针4字节
受害者IP地址4字节
\t分割的UTF-8字符，数据结构为ComputerName\tUsername\tBeaconProcessName

### 公钥/私钥生成和提取
AES密钥是通过非对称密钥加密的，很难破译。
C2通信时通过对称密钥加密的，很难找到指纹标记。
### 3.C2流量分析
流量分析分为三个部分：
1. Cobalt Strike Beacon 下载
2. C2 Beacon心跳
   对心跳包的Cookie进行解密（运行cs-crypto-parser.py 脚本）拿到RSA的public Key 和private key
4. C2任务请求和响应
   有效负载的Cookie继续进行解密拿到Sha256digest和AESkey HMACkey

解密过程由Decrypt()函数处理，该函数执行以下操作：
1.从加密的有效负载中提取 HMAC 签名。
2.使用加密负载上的 HMAC 密钥计算并验证 HMAC 签名。
3.加载 AES 密钥并设置模式 (CBC) 及其初始化向量 (IV)。
4.解密加密的有效负载。
当Beacon接收并执行C2服务器提供的任务时，结果被收集并返回给TeamServer。
## cobalt-strike配置文件
![image](https://github.com/kyre0e/kyre0e.github.io/assets/169347540/89b5551a-d873-4185-94e7-04feca214a50)
1.对于 GET 请求，大多数请求 URI 都非常短并且具有预定义的模式。 这些 URI 是从图 1 中默认配置文件中的 set uri 下指定的 URI 列表中随机选择的。 如果恶意攻击者使用在 http-get 部分内设置了 uri 选项的自定义配置文件，则可以轻松地将 URI 修改为任意字符串。 基于模式的签名可以很好地捕获使用默认配置文件的 Cobalt Strike 流量，但无法捕获自定义配置文件的任何变化。
2.对于 POST 请求，URI 中有一个预定义模式 – /submit.php?id=。 ID值是随机生成的。 与 HTTP GET 请求的可能性类似，如果恶意攻击者使用在 http-post 部分中设置了 uri 选项的自定义配置文件，则恶意攻击者可以轻松地将 URI 修改为任意字符串。

