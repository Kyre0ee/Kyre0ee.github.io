---
layout: post
title: 7.Xposed模块编写&Api详解&快速Hook
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
`
Class a = loadPackageParam.classLoader.loadClass("com.zj.wuaipojie.Demo");
XposedBridge.hookAllMethods(a, "Inner", new XC_MethodHook() {
    @Override
    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
        super.beforeHookedMethod(param);
        }
});`
3. Hook替换函数
`class a = classLoader.loadClass("类名")
 XposedBridge.hookAllmethods(a,"getId",new XC_MethodReplacement() {
     @Override
     protected Object replaceHookedMethod(MethodHookParam methodHookParam) throws Throwable {
         return "";
     }
});`
4. Hook加固通杀
`XposedHelpers.findAndHookMethod(Application.class, "attach", Context.class, new XC_MethodHook() {
     @Override
     protected void afterHookedMethod(MethodHookParam param) throws Throwable {
          Context context = (Context) param.args[0];
          ClassLoader classLoader = context.getClassLoader();//先拿到加载器classLoader
          //做后续操作
   }
});`
# 7.Xposed常用API
***
## 7.1 Hook变量
静态变量与实例变量：
+ 静态变量（static）:类被初始化，同步进行初始化
+ 非静态变量：类被实例化（产生一个对象的时候），进行初始化
  `//静态变量
  final Class clazz = XposedHelpers.findClass("类名", classLoader);
  XposedHelpers.setstaticIntField(clazz, "变量名", 999);
  //实例变量
  final Class clazz = XposedHelpers.findClass("类名", classLoader);
  XposedBridge.hookAllConstructors(clazz, new XC_MethodHook() {
      @Override
      protected void afterHookedMethod(MethodHookParam param) throws Throwable {
          super.afterHookedMethod(param);
          //param.thisObject获取当前所属的对象
          Object ob = param.thisobject;
          XposedHelpers.setIntField(ob,"变量名",999);
  )`
## 7.2 Hook构造函数
`//无参数构造函数
 XposedHelpers.findAndHookConstructor("com.zj.wuaipojie.Demo", classLoader, new XC_MethodHook() {
     @Override
     protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
         super.beforeHookedMethod(param);
     }
     @Override
     protected void afterHookedMethod(MethodHookParam param) throws Throwable {
         super.afterHookedMethod(param);
     }
});
//有参构造函数
XposedHelpers.findAndHookConstructor("com.zj.wuaipojie.Demo", classLoader, String.class,new XC_MethodHook() {
     @Override
     protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
         super.beforeHookedMethod(param);
     }
     @Override
     protected void afterHookedMethod(MethodHookParam param) throws Throwable {
         super.afterHookedMethod(param);
     }
});
`
## 7.3 HookmultiDex方法
`
XposedHelpers.findAndHookMethod(Application.class, "attach", Context.class, new XC_MethodHook() {
    @Override
    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
        ClassLoader c1 = ((Context)param.args[0]).getClassLoader();
        Class<?> hookclass = null;
        try {
            hookclass=c1.loadClass("类名");
        }catch (Exception e){
            Log.e("zj2595","未找到类", e);
            return;
        }
        XposedHelpers.findAndHookMethod(hookclass, "方法名", new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
            }
        });
    }
});
`
## 7.4 主动调用
`
//静态方法
Class clazz = XposedHelpers.findClass("类名",lpparam.classLoader);
XposedHelpers.callStaticMethod(clazz,"方法名",参数(非必须))；
//实例方法
Class clazz = XposeHelpers.findClass("类名",lpparam.classLoader);
XposedHelpers.callMethod(clazz.newInstance(),"方法名",参数(非必须))；
`
## 7.5 Hook内部类
`内部类:类里还有一个类class
XposedHelpers.findAndHookMethod("com.zj.wuaipojie.Demo$InnerClass", lpparam.classLoader, "innerFunc", String.class, new XC_MethodHook()) {
    @Override
    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
        super.beforeHookedMethod(param);
    }
  });`
## 7.6 反射大法
`
Class clazz = XposedHelers.findClass("com.zj.wuaipojie.Demo", lpparam.classLoader);
XposedHelpers.findAndHookMethod("com.zj.wuaipojie.Demo$InnerClass", lpparam.classLoader, "innerFunc",String.class, new XC_MethodHook()){
    @Override
    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
        super.beforeHookMethod(param);
        //第一步找到类
        //找到方法，如果是私有方法就要setAccessible设置访问权限
        //invoke主动调用或者set修改值(变量)
        Class democlass = Class.forName("com.zj.wuaipojie.Demo",false,lpparam.classLoader);
        Method demomethod = democlass.getDeclaredMethod("refl");
        demomethod.setAccessible(true);
        demomethod.invoke(clazz.newInstance());
    }
});
`
## 7.7 遍历所有类下的所有方法
`
XposedHelpers.findAndHookMethod(ClassLoader.class, "loadClass", String.class, new XC_MethodHook() {
    @Override
    protected void afterHookMethod(MethodHookParam param) throws Throwable {
        supper.afterHookedMethod(param);
        Class clazz = (Class) param.getResult();
        String clazzName = clazz.getName();
        //排除非包名的类
        if(clazzName.contains("com.zj.wuaipojie")){
            Method[] mds = clazz.getDeclaredMethods();
            for(int i = 0;i<mds.length;i++){
                final Method md = mds[i];
                int mod = mds[i].getModifiers();
                //去除抽象、native、接口方法
                if(!Modifier.isAbstract(mod)&& !Modifier.isNative(mod) &&!Modifier.isAbstract(mod)){
                    XposedBridge.hookMethod(mds[i], new XC_MethodHook() {
                    @Override
                    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                        super.beforeHookedMethod(param);
                        Log.d("zj2595",md.toString());
                    }
              });
         }
  }
}
}
});
`
## 7.8 Xposed妙用
`
//字符串赋值定位
XposedHelpers.findAndHookMethod("android.widget.TextView",lpparam.classLoader, "setText", CharSqeuence.class, new XC_MethodHook() {
    @Override
    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
        super.beforeHookedMethod(param);
        Log.d("zj2595",param.args[0].toString());
        if(param.args[0].equals("已过期")){
            printStackTrace();
        }
    }
});
//打印堆栈调用代码的实现
pricate static void printStackTrace() {
    Throwable ex = new Throwable();
    StackTraceElement[] stackElements = ex.getStackTrace();
    for(int i = 0; i < stackElements.length; i++) {
        StackTraceElement element = stackElements[i];
        Log.d("zj2595","at "+element.getClassName()+"."+element.getMethodName() + "(" + element.getFileName())
    }
}
//点击事件监听
Class clazz = XposedHelpers.findClass("android.view.View", lpparam.classLoader);
XposedBridge.hookAllMethods(clazz, "perfformClick", new XC_MethodHook() {
    @Override
    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
        super.afterHookedMethod(param);
        Object listenerInfoObject = XposedHelpers.getobjectField(param.thisObject,"mListenerInfo");
        Object mOnclickListenerObject = XposedHelpers.getObjectField(listenerInfoObject, "mOnClickListener");
        String callbackType = mOnClickListenerObject.getClass().getName();
        Log.d("zj2595",callbackType);
    }
  });
//改写布局
XposedHelpers.findAndHookMethod("com.zj.wuaipojie.ui.ChallengeSixth",lpparam.classLoader, "onCreate", Bundle.class, new XC_MethodHook() {
    @Override
    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
        super.afterHookedMethod(param);
        View img = (View)XposedHelpers.callMethod(param.thisObject, "findViewById",0x7f0800de);//传入要找到的id的十六进制
        img.setVisibility(View.GONE);
        }
    }
});
`
# 8.Xposed模块patch
LSPatch
# 9.Xposed快速Hook
SimpleHook
jsHook
