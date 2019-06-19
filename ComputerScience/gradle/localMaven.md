### 本地仓库  
minePluginUri = file:///Users/alex/WorkSpace/Gradle/MinePlugin/repo/  

provide library  
项目 gradle  
```
buildscript {
    repositories {
        maven { url minePluginUri }
    }    
}
allprojects {
    repositories {
        maven { url minePluginUri }
    }
}
```
module 的 gradle  
```
apply plugin: 'com.android.library'
apply plugin: 'maven'
android {
    compileSdkVersion 27
    defaultConfig {
    }
    buildTypes {
    }
}
dependencies {
}

uploadArchives {
    repositories {
        mavenDeployer {
            //设置插件的GAV参数
            pom.groupId = 'org.alex.basic'
            pom.artifactId = 'basicLibrary'
            pom.version = '1.0.0'
            //文件发布到下面目录
            repository(url: uri(minePluginUri))
        }
    }
}

```
using library  
项目 gradle  
```
buildscript {
    repositories {
        maven { url minePluginUri }
    }    
}
allprojects {
    repositories {
        maven { url minePluginUri }
    }
}
```
module 的 gradle  
```
apply plugin: 'com.android.application'
android {
    compileSdkVersion 27
    defaultConfig {
     }
    buildTypes {
    }
}

dependencies {
    compile 'org.alex.basic:basicLibrary:1.0.0@aar'
}

```
### 常见错误  
android发布maven报错:Could not write to file '*****/build/poms/pom-default.xml'  
拆包 和 apt 代码, 都是不行的;  
```
android {
    //...
    defaultConfig {
        //...
        multiDexEnabled true
    }
    
   或者
    
   dataBinding {
    //  enabled = true
    }
    
}
```
### 打包与发布的问题  
```
Error:Could not find PreferenceAnnotion:preference-annotation:unspecified.  
```
例如我有: platform, player, player-skin 这几个库;  
在 player build.gradle 是这样配置的  
```
    if(isLibPlatformSource.toBoolean()){
        implementation project(':library_platform')
        logV("hello library_platform 源码依赖")
    }else{
        implementation libs.gsl_platform
        logV("hello ${libs.gsl_platform}")
    }
```
必须改成  
```
implementation libs.gsl_platform
```
暂时不知道, 为什么  

### 参考   
http://kvh.io/cn/embrace-android-studio-maven-deploy.html  
https://github.com/JeasonWong/CostTime/    
