插件 version 与 gradle version 的对应关系  
Plugin version         Required Gradle version  
1.0.0 - 1.1.3                 2.2.1 - 2.3  
1.2.0 - 1.3.1                 2.2.1 - 2.9  
1.5.0                              2.2.1 - 2.13  
2.0.0 - 2.1.2                 2.10 - 2.13  
2.1.3 - 2.2.3                 2.14.1+  
2.3.0+                            3.3+  
3.0.0+                            4.1+   
3.1.0+                            4.4+  
3.2.0 - 3.2.1                 4.6+  
3.3.0 - 3.3.2                 4.10.1+  


例如:  
gradle plugin 4.5 对应 classpath 'com.android.tools.build:gradle:3.1.0'  


```
//  gradle/wrapper/gradle-wrapper.properties  

distributionUrl=file\:/Users/alex/WorkSpace/Gradle/gradle-3.3-all.zip  
```

### sdkVersion  
三者关系:  
minSdkVersion <= targetSdkVersion <= compileSdkVersion;  
理想状态:  
minSdkVersion <= targetSdkVersion == compileSdkVersion;  

compileSdkVersion 告诉 Gradle 用哪个版本的 SDK 来编译;  
修改 compileSdkVersion 不会改变运行时的行为, 可能会出现新的编译警告, 编译错误等;  
但新的 compileSdkVersion 不会被包含到 APK 中, 它只是在编译的时候使用, 因此我们强烈推荐总是使用最新的 SDK 进行编译;  

minSdkVersion 是应用可以运行的最低要求;  
是商店用来判断, 用户设备是否可以, 安装某个应用的标志之一;  
minSdkVersion 是一个商业决策问题, 一般支持 97%以上, 额外的支持会带来更多的开发和测试成本;  

targetSdkVersion 是 Android 提供向前兼容的主要依据;  
如果 targetSdkVersion 为19 对应 Android4.4, 应用运行时, 最高只能使用API 19的新特性;  
即使代码中使用了API 23的新特性, 实际运行时, 也不会使用该新特性;  

### gradle日志  
ERROR	          错误消息  
QUIET	              重要消息  
WARNING	      警告消息  
LIFECYCLE	      进度消息  
INFO	              信息消息  
DEBUG	          调试信息  

-s 或者 –stacktrace	          输出关键性的堆栈信息  
-S 或者 –full-stacktrace	  输出全部堆栈信息  

//  输出QUIET级别及其之上的日志信息
gradle -q tasks  
//  输出INFO级别及其之上的日志信息
gradle -i tasks  


