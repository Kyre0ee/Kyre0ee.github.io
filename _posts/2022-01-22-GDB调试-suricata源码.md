## GDB调试-suricata源码

### 1.   静态分析

函数调用关系是：DecodeEthernet->DecodeIPV4->DecodeIPV4Packet

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

### 2.动态调试

1.将断点下在DecodeEthernet函数处

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)

2.继续运行，断在该函数处

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image006.jpg)

2.     单步执行调用了DecodeIPV4()函数

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image008.jpg)

3.步入DecodeIPV4函数

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image010.jpg)

4.单步执行调用了DecodeIPV4Packet函数

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image012.jpg)

5.步入DecodeIPV4Packet函数

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image014.jpg)

1.     包大小小于1500时

此时的len = 获取到的包长度，不会给包进行一个截断包的标识

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image016.jpg)

2.     继续运行，可以看到包大小小于1500时，获取len是等于包大小的。当包大小为1514时获取到的包大小是1500.

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image018.jpg)

3.     继续执行前面的调用过程

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image020.jpg)

10.此时RAX寄存器中的内容大于RBP寄存器中的内容，满足判断条件为真，为该pcap包打上截断包的标志。

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image022.jpg)

![](file:///C:/Users/zhukeyu/AppData/Local/Temp/msohtmlclip1/01/clip_image024.jpg)