### 代码混淆  
### gradle 配置  
```

android {
    compileSdkVersion 28
    defaultConfig {
        
    }
    def isMinifyEnabled = true
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
        debug {
            debuggable true
            //Zipalign优化
            zipAlignEnabled isMinifyEnabled
            // 移除无用的resource文件
            shrinkResources isMinifyEnabled
            //混淆--
            minifyEnabled isMinifyEnabled
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```
[基础模板](library/template_01.md)  

keep 某个类  
```
-keep class androidx.versionedparcelable.VersionedParcel { *; }  
```

keep 某个包  
```
-dontwarn android.support.design.**
-keep class android.support.design.** { *; }
```



### 参考  
https://github.com/xitu/gold-miner/blob/master/TODO/troubleshooting-proguard-issues-on-android.md  
https://github.com/krschultz/android-proguard-snippets/tree/master/libraries  
http://ljfxyj2008.github.io/2015/12/22/Android%E4%B8%AD%E7%9A%84Proguard.html  
