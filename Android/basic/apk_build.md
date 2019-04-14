###  apk打包流程  
通过 aapt 打包 res 资源文件, 生成 R.java 文件, 二进制的 resources.arsc 文件, 和 res 文件;  
res 文件包括: res/raw 文件, 图片资源文件, 他们保持原状, 会生成资源 id,  定义在 R.java 文件中;  

aidl 工具会将所有的 aidl 接口转化为 .java 文件;  
javac 会把所有的 .java 文件编译成 .class 文件;  
然后切入 proguard, 做代码优化和混淆;  
dex 工具会将上述产生的 .class 文件, 编译成 .dex 文件;  
apkBuilder 工具会把 resources.arsc, res 文件, assets 文件, 以及 .dex 文件, .so 文件, 打包成 apk 文件;  
通过 Jarsigner 工具, 对 apk 进行 debug 或 release 签名;  

无论是 debug 还是 release , 都会使用 zipalign 工具对 APK 进行对齐操作;  
使 apk 中所有资源文件, 距离文件起始偏移为4字节的整数倍, 从而在通过内存映射访问 apk 文件时会更快, 当应用运行时会减少内存的开销;  

❀ 为什么, 这些 XML 资源文件, 要从文本格式编译成二进制格式?   
解决了空间占用以及解析效率的问题;  
1.. 二进制格式的 XML 文件, 占用空间更小;  
这是由于, 所有 XML 元素的标签, 属性名称, 属性值和内容, 所涉及到的字符串, 都会被统一收集到, 一个字符串资源池中去, 并且会去重;  
有了这个字符串资源池, 原来使用字符串的地方, 就会被替换成, 字符串资源池的索引值, 是一个 int 类型, 从而可以减少文件的大小;  
2.. 二进制格式的XML文件解析速度更快;  
这是由于二进制格式的 XML 元素里面, 不再包含有字符串值, 因此就避免了进行字符串解析, 从而提高速度;  

Android资源管理框架, 的另外一个重要任务, 就是要根据资源ID, 来快速找到对应的资源;  

### overlay#重叠包  
❀ 表示当前正在编译的资源的重叠包, 重叠包是什么概念呢?   举个例子:  
```
aapt package \
-M AndroidManifest.xml \
-m -J gen \
-S src/com/example/res \
-S src/com/example/ui/res
```
假如我们有如上的 aapt 命令输入, 那么当 src/com/example/res 与src/com/example/ui/res 有相同资源的时候, 就会使用前者, 因为前者已经生成并存在了;  
这里对资源替换的粒度是 resource 而不是文件, 比如两个文件夹的 values/string.xml 都有对同一个 string id 的描述, 最后就会使用前者的字符串; 

然后我们再来看看 --auto-add-overlay 有什么用, 
假如我们在 src/com/example/ui/res 定义了资源 string a, 但是在 src/com/example/res 却没有这个string, 那就会报错, 因为基础包里是没有那个资源的;  
这时候就需要加上 --auto-add-overlay, 于是就会自动把新的资源都添加进去;  
❀ 第二个例子  
```
res   
└── values
│       ├── colors.xml
│       ├── dimens.xml
│       ├── strings.xml
│       └── styles.xml    //  hello, world, 引用了 hehe  
res2
│   └── values
│       └── strings.xml    //  hehe res2 
└── res3
    └── values
        └── strings.xml   //  hehe res3  
```
res2 和 res3 分别定义了一个 string hehe, value 分别为 hehe res2 和 hehe res3;  
```
android {
  ...
  aaptOptions {
      additionalParameters '-S',
              '/Users/yifan/dev/github/Testapp/app/src/main/res3',
              '-S',
              '/Users/yifan/dev/github/Testapp/app/src/main/res2',
              '--auto-add-overlay'
      noCompress 'foo', 'bar'
      ignoreAssetsPattern '!.svn:!.git:!.ds_store:!*.scc:.*:<dir>_*:!CVS:!thumbs.db:!picasa.ini:!*~'
  }
  ...
}
```
在屏幕中央, 显示了hehe res3, 交换 -S 顺序后则变成了hehe res2, 符合我们第一节中说到的, 选择首个匹配原则;  
所有的资源都可以使用重叠包来进行动态指定;  



### 65535问题  

app gradle  
```
android {

    defaultConfig {
        multiDexEnabled = true
    }
    
}
```


Application  
```
override fun attachBaseContext(base: Context?) {
    super.attachBaseContext(base)
    MultiDex.install(base)
}
```

MultiDex  
因为在Dalvik指令集里，调用方法的invoke-kind指令中，method reference index只给了16bits，最多能调用65535个方法，  
所以在生成dex文件的过程中，  当方法数超过65535就会报错。细看指令集，除了method，field和class的index也是16bits，所以也存在65535的问题。  
一般来说，method的数目会比field和class多，所以method数会首先遇到65535问题，你可能都没机会见到field过65535的情况。  
为 Dalvik 可执行文件分包构建每个 DEX 文件时，构建工具会执行复杂的决策制定来确定主要 DEX 文件中需要的类，以便应用能够成功启动。  
如果启动期间需要的任何类未在主 DEX 文件中提供，那么您的应用将崩溃并出现错误 java.lang.NoClassDefFoundError。  
该情况不应出现在直接从应用代码访问的代码上，因为构建工具能识别这些代码路径，但可能在代码路径可见性较低（如使用的库具有复杂的依赖项）时出现。  
例如，如果代码使用自检机制或从原生代码调用 Java 方法，那么这些类可能不会被识别为主 DEX 文件中的必需项。    
因此，如果您收到 java.lang.NoClassDefFoundError，则必须使用构建类型中的 multiDexKeepFile 或 multiDexKeepProguard 属性声明它们，  
以手动将这些其他类指定为主 DEX 文件中的必需项。如果类在 multiDexKeepFile 或 multiDexKeepProguard 文件中匹配，则该类会添加至主 DEX 文件。  

multiDexKeepFile 属性  
您在 multiDexKeepFile 中指定的文件应该每行包含一个类，并且采用 com/example/MyClass.class 的格式。  
例如，您可以创建一个名为 multidex-config.txt 的文件，如下所示：  
```
com/example/MyClass.class
com/example/MyOtherClass.class
```
然后，您可以按以下方式针对构建类型声明该文件：  
```
android {
    buildTypes {
        release {
            multiDexKeepFile file 'multidex-config.txt'
            ...
        }
    }
}
```
请记住，Gradle 会读取相对于 build.gradle 文件的路径，因此如果 multidex-config.txt 与 build.gradle 文件在同一目录中，以上示例将有效。  


multiDexKeepProguard 属性  
multiDexKeepProguard 文件使用与 Proguard 相同的格式，并且支持整个 Proguard 语法。  
您在 multiDexKeepProguard 中指定的文件应该在任何有效的 ProGuard 语法中包含 -keep 选项。  
例如，-keep com.example.MyClass.class。您可以创建一个名为 multidex-config.pro 的文件，如下所示：  
```
-keep class com.example.MyClass
-keep class com.example.MyClassToo
```
如果您想要指定包中的所有类，文件将如下所示：  
```
-keep class com.example.** { *; } // All classes in the com.example package
```
然后，您可以按以下方式针对构建类型声明该文件：  
```
android {
    buildTypes {
        release {
            multiDexKeepProguard 'multidex-config.pro'
            ...
        }
    }
}
```


###  参考 
http://mouxuejie.com/blog/2016-08-04/build-and-package-flow-introduction/  
https://blog.csdn.net/jiangwei0910410003/article/details/50628894  
https://juejin.im/entry/58b78d1b61ff4b006cd47e5b  
http://blog.zhaiyifan.cn/2016/02/13/android-reverse-2  
https://blog.csdn.net/luoshengyang/article/details/8744683  

65535 问题  
http://jiajixin.cn/2015/10/21/field-65535/  
http://www.jianshu.com/p/33f22b21ef1e  
https://developer.android.com/studio/build/multidex  
https://www.jianshu.com/p/a85bc59d6549   


dex              Dalvik Executable        
AAPT          Android Asset Packaging Tool
javac           Java Compiler                                                 java 编译器  
mmap        内存映射  
