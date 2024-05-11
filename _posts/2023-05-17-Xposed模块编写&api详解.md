---
layout: post
title: 7.Xposed模块编写&Api详解
date: 2023-05-17 15:57 +0800
tags: 安卓逆向
toc: true
---
# 1.什么是Xposed?
***
xposed是一款可以在不修改APK的情况下影响程序运行的框架，基于它可以制作出许多功能强大的模块，且在功能不冲突的情况下同时运行。在这个框架下，我们可以编写并加载自己编写的插件app,实现对目标apk的注入拦截等。
# 2.Xposed原理
***
用自己实现的app_process替换掉了系统原本提供的app_process，加载一个额外的jar包，入口从原来的：com.android.internal.osZygoteInit.main()被替换成了：de.robv.android.xposed.XposedBridge.main(),创建的Zygote进程就变成Hook的Zygote进程了，从而完成对zygote进程及其创建的Dalvik/ART虚拟机的劫持（zytoge注入）
> Android系统启动--->zygote启动---->从zygote fork子进程（对应进程文件app_process）
> 安装xposed--->替换新的app_process至/system/bin/目录下--->替换虚拟机--->zygote启动--->加载xposedBridge.jar至系统目录--->启动Xposed替换的虚拟机
# 3.Xposed可以做什么？
***
1. 修改app布局：开启上帝模式
2. 劫持数据，修改参数值、返回值、主动调用等。例：微信防撤回、步数修改、一键新机
3. 自动化操作，例：微信自动抢红包
# 4.xposed环境配置
***
1. ubuntu虚拟机：Frida开发环境、动态分析及开发工具android-studio、动态分析工具ddms、静态分析工具：jadx1.4.4、动静态分析工具jeb、动态分析工具集成hyperpwn、静态分析工具010editor、抓包工具charles、抓包工具wireshark、动态分析工具unidbg
> 1.Android Studio创建新项目
> 2.将下载的xposedBridgeApi.jar包拖进libs文件夹
> 3.点击jar包，选择add as library
> 4.修改xml文件配置，在application中添加
`<!-- 是否为xposed模块，xposed根据这个来判断是否是模块-->
<meta-data android:name="xposedmodule" android:value="true"/>
<!-- 模块描述，显示在xposed模块列表第二行-->
<meta-data android:name="xposeddescription" android:value="这是一个xposed模块"/>
<!-- 最低xposed版本号(lib文件名可知)-->
<meta-data android:name="xposedminversion" android:value="89"/>
`
> 5.修改build.gradle,将此处修改为compileOnly默认的是implementation
`implementation使用该方式依赖的库将会参与编译和打包
compileOnly 只在编译时有效，不会参与打包`
> 6.新建-->Folder-->Assets Folder，创建xposed_init(不要后缀名)：只有一行代码，就是说明入口类
> 7.新建Hook类，实现IxposedHookLoadPackage接口，然后在handleLoadPackage函数内编写Hook逻辑,使其继承IXposedHookLoadPackage拥有hook的能力。
`import de.robv.android.xposed.IXposeHookLoadPackage;
 import de.robv.android.xposed.callbacks.XC_LoadPackage;
 public calss Hook implements IXposedHookLoadPackage {
     @Override
     public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
    }
}`
# 6.Xposed常用API
1. Hook普通方法
   * 修改返回值
`XposedHelpers.findAndHookMethod("com.zj.wuaipojie.Demo", classLoader, "getPublicInt", new XC_MethodHook(){
     @Override
     protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
         super.beforeHookedMethod(param);
     }
     @Override
     protected void afterHookedMethod(MethodHookparam param) throws Throwable {
         super.afterHookedMethod(param);
         param.setResult(999);
     }
});`
   * 修改参数
`XposedHelpers.findAndHookMethod("com.zj.wuaipojie.Demo", classLoader, "setPublicInt", int.class,new XC_MethodHook(){
     @Override
     protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
         super.beforeHookedMethod(param);
         param.args[0] = 999;
     }
     @Override
     protected void afterHookedMethod(MethodHookparam param) throws Throwable {
         super.afterHookedMethod(param);
     }
});`

2. Hook复杂&自定义参数
`XposedHelpers.findAndHookMethod("com.zj.wuaipojie.Demo", classLoader, "complexParameterFunc", java.lang.String.class,java.util.HashMap,new XC_MethodHook(){
     @Override
     protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
         super.beforeHookedMethod(param);
         param.args[0] = 999;
     }
     @Override
     protected void afterHookedMethod(MethodHookparam param) throws Throwable {
         super.afterHookedMethod(param);
     }
});`
