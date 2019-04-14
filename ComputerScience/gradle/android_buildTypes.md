### applicationId添加后缀  
```
android {
    buildTypes {
        debug {
            applicationIdSuffix ".debug"
        }

        jnidebug.initWith(buildTypes.debug)
        jnidebug {
            packageNameSuffix ".jnidebug"
            jniDebuggable true
        }
    }
}
```
### BuildConfig添加字段  
java  
```
public final class BuildConfig {
  // Fields from build type: debug
  public static final boolean IS_NEED_IPC = true;
  public static final boolean LOG_ENABLE = true;
  public static final String SERVER_HOST = "http://200.200.200.50:8080/";
}
```
gradle  
```
android {
    buildTypes {
        debug {
            buildConfigField("boolean", "IS_NEED_IPC", "true")
            buildConfigField("boolean", "LOG_ENABLE", "true")
            buildConfigField "String", "SERVER_HOST", quotation("http://200.200.200.50:8080/")
        }
        release {
            buildConfigField("boolean", "IS_NEED_IPC", "true")
            buildConfigField("boolean", "LOG_ENABLE", "true")
            buildConfigField "String", "SERVER_HOST", quotation("http://200.200.200.60:8080/")
        }
    }
}
```

### 配置签名工具     
```
android {
    ...
    defaultConfig {...}
    signingConfigs {
        release {
            storeFile file("myreleasekey.keystore")
            storePassword "password"
            keyAlias "MyReleaseKey"
            keyPassword "password"
        }
    }
    buildTypes {
        release {
            ...
            signingConfig signingConfigs.release
        }
    }
}
```

### 关于ndk  
```
android {
    defaultConfig {
    
    //配置ndk的一些规则
    externalNativeBuild {
        cmake {
            cppFlags "-frtti -fexceptions"
        }
        //配置Gradle 构建时需要的.so动态库
        // Gradle会构建那些 ABI 配置,但是只会将 defaultConfig.ndk {} 代码块中指定的配置打包到 apk 中
        ndkBuild{
            abiFilters 'armeabi','armeabi-v7a'
        }
    }
    
    //  指打包到 apk 里面的.so 包种类
    ndk{
        //  选择要添加的对应cpu类型的.so库;
        abiFilters 'armeabi', 'armeabi-v7a', 'x86' // 还可以添加 'armeabi-v8a', 'x86_64', 'mips', 'mips64'
    }
}

```
mips / mips64: 极少用于手机可以忽略;  
x86 / x86_64: 极少用于手机可以忽略, 但是模拟器可能会用;  
armeabi: ARM v5 这是相当老旧的一个版本, 缺少对浮点数计算的硬件支持, 在需要大量计算时有性能瓶颈;  
armeabi-v7a: ARM v7 目前主流版本;  
arm64-v8a: 64位支持;  
arm64-v8a 是可以向下兼容的, 其下有armeabi-v7a, armeabi;  
armeabi-v7a 向下兼容 armeabi;  
目前市面上主流手机cpu多属于armeabi-v7a;  

为了减小 apk 体积, 只保留 armeabi 和 armeabi-v7a 两个文件夹, 并保证这两个文件夹中 .so 数量一致;  
对只提供 armeabi 版本的第三方 .so , 原样复制一份到 armeabi-v7a 文件夹;  


### splits   
```
android {
  splits {
      density {
          //  true 表示打开或关闭APK分割功能;  
          enable true
          //  若要使用include功能, 则使用前需调用reset();  
          reset()
          //  创建白名单, 仅构建出白名单中指定的格式
          include ''
          //  不会构建出黑名单中指定的格式;  
          exclude 'ldpi', 'mdpi'
          compatibleScreens 'normal', 'large', 'xlarge'
          //  默认为true, 即除了指定的格式外, 还会构建出一个通用的APK;  
          universalApk false      
    }
    abi {
          enable true
          reset()
          include 'x86', 'armeabi-v7a', 'mips'
          universalApk false
      }
  } 
}
生成结果：
app-hdpi-release.apk
app-universal-release.apk
app-xhdpi-release.apk
app-xxhdpi-release.apk
app-xxxhdpi-release.apk
```
### defaultConfig  
```
android {
    defaultConfig {
        resConfigs 'en', 'zh-rCN', 'xxhdpi'
        ndk {
            abiFilters 'armeabi', 'armeabi-v7a'
        }
    }
}    
```
其中, android.defaultConfig.ndk 和 android.splits 有冲突;  
```
ERROR: Conflicting configuration : 'armeabi-v7a,armeabi' in ndk abiFilters cannot be present when splits abi filters are set : armeabi-v7a,armeabi
```
所以, 要想使用 android.defaultConfig.ndk, 就必须使得 
android.splits.abi.enable = false;  
android.splits.density.enable = false;  

### 加快编译速度, 优化apk大小  
```
android {
    defaultConfig {
        if (useFastBuild) {
            resConfigs 'en', 'zh-rCN', 'xxhdpi'
        }
        ndk{
            //  默认这两种, 即可
            abiFilters 'armeabi', 'armeabi-v7a'
        }
    }
    dexOptions {
        dexInProcess true
        //  预编译, 增量编译更快, 但是会导致 clean 变慢;
        preDexLibraries true
        maxProcessCount 8
        threadCount = 16
        javaMaxHeapSize "4g"
        // 忽略方法数限制的检查, 支持大工程模式;
        jumboMode true
        javaMaxHeapSize "1280m"
    }
    aaptOptions {
        //  关闭Android Studio的PNG合法性检查, 停用 PNG 处理;  
        //  不需要将 PNG 图像转换成 WebP, 加快构建速度, 要停用此优化;  
        //  在版本发布的时候, 需要设为 true;  
        if (useFastBuild) {
            cruncherEnabled = false
            useNewCruncher = false
        }
    }
    
    buildTypes {
          debug {
                if (useFastBuild) {
                    //  关闭 Crashlytics 报告  
                    ext.enableCrashlytics = false
                    ext.alwaysUpdateBuildId = false
                    zipAlignEnabled false
                    //  禁止拆分多个apk文件, 速度会更快
                    splits.abi.enable = false
                    splits.density.enable = false
                }
                proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            }
        }
    }
    
    productFlavors { 
            //  快速编译版
            fastBuild {
                def name = productFlavors[1].name
                def nameFormat = quotation(name)
                manifestPlaceholders = [
                        "APP_LABEL": "学习Gradle($name)"
                ]
                dimension 'BUSINESS'
                //  遇到 64k方法限制, 21以上避免 Legacy Multidex, 构建更快
                minSdkVersion '21'
                resConfigs('en', 'xxhdpi')
                buildConfigField "String", "FLAVORE_NAME", nameFormat
                versionNameSuffix "-$name"
            } 
        }
}
```
❀ 修改 gradle.properties  
```
org.gradle.jvmargs=-Xmx3072m -XX:MaxPermSize=1024m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
#  开启守护线程
org.gradle.daemon=true
#  开启并行编译任务
org.gradle.parallel=true
#  加快多模块构建
org.gradle.configureondemand=true
org.gradle.caching=true

#  开启缓存
android.enableBuildCache=true
#  新版编译器(D8), 加快构建速度
android.enableD8=true
#  禁止 check, 尽快构建速度
android.enableBuildScriptClasspathCheck=false
```
❀ 修改 studio.exe.vmoptions  
```
-server
-Xms2048m
-Xmx2048m
-XX:MaxPermSize=2048m
-XX:ReservedCodeCacheSize=1024m
-XX:+UseConcMarkSweepGC
-XX:SoftRefLRUPolicyMSPerMB=50
-XX:+HeapDumpOnOutOfMemoryError
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-Djdk.http.auth.tunneling.disabledSchemes=""
-Djna.nosys=true
-Djna.boot.library.path=
-Djna.debug_load=true
-Djna.debug_load.jna=true
-da

-Didea.platform.prefix=AndroidStudio
```
修改 module gradle 文件  
```

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
}

if (useFastBuild) {
    tasks.whenTaskAdded { task ->
        if (task.name.contains('AndroidTest') || task.name.contains('Test')) {
            task.enabled = false
        }
    }
}

if (ignoreLintCheck) {
    tasks.whenTaskAdded { task ->
        if (task.name.equalsIgnoreCase("lint") || task.name.equalsIgnoreCase("lintVitalRelease")) {
            task.enabled = false
        }
    }
}
```
### 参考  
https://zhuanlan.zhihu.com/p/21359984  
https://blog.csdn.net/ouyang_peng/article/details/51168072  
https://www.jianshu.com/p/5dd1d8db8f2c  
