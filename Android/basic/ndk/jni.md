1.. 新建项目, 包含 C++环境;  
2.. 新建java类  
```
public class JniHelper {
    static {
	   /* moduleName  "HelloJni"  */
        System.loadLibrary("alex-jni-fun");
    }
    /**
     * 从 jni-c 文件中获取 字符串，这时候有红色，不用管
     */
    public static native String getStringFromC();
}
```
2.. rebuild  
Build -> rebuild project  


### cmake  
配置 app/build.gradle  
```
android {
    defaultConfig {
        //  有两个 abiFilters, 暂时不清楚区别, 先都保留  
        externalNativeBuild {
            cmake {
                cppFlags ""
                abiFilters 'arm64-v8a', 'armeabi-v7a'
            }
        }

        ndk {
            abiFilters 'arm64-v8a', 'armeabi-v7a'
        }
    }
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
            version "3.10.2"
        }
    }
    buildTypes {    }
}
dependencies {  }
```
新建一个  app/CMakeLists.txt  
sync 一下, 再看 studio.Build 目录下, 多了 Refresh Linked C++ Projects  
```
cmake_minimum_required(VERSION 3.4.1)  
add_library( 
            # 生成 libalex-jni-fun.so 文件
             native-lib
             # STATIC  静态库, 是目标文件的归档文件, 在链接其它目标的时候使用
             # SHARED  动态库, 会被动态链接, 在运行时被加载
             # MODULE  模块库, 是不会被链接到其它目标中的插件, 但是可能会在运行时使用dlopen-系列的函数动态链接
             SHARED

             # 资源文件，可以多个，
             # 资源路径是相对路径，相对于本CMakeLists.txt所在目录
            HelloFun.cpp
            LogTrack.cpp)
             
# 从系统查找依赖库
find_library( 
              # android 系统每个类型的库会存放一个特定的位置, 而log库存放在log-lib中
              log-lib

              # android系统在c环境下打log到logcat的库
              log )

# 配置库的链接（依赖关系）
target_link_libraries( 
                       # 引用 libalex-jni-fun.so 文件
                       alex-jni-fun
                       # 依赖于
                       ${log-lib} )              
``` 
### 示例代码  
![示例1](sample/sample001.md)  

打印日志  
```
#include "android/log.h" 
#define LOGD(...) __android_log_print(ANDROID_LOG_DEBUG , "ffmpeg.c", __VA_ARGS__) 
#define LOGE(...) __android_log_print(ANDROID_LOG_ERROR , "ffmpeg.c", __VA_ARGS__)
```
### 参考  
cmake demo  
https://blog.csdn.net/you__are_my_sunshine/article/details/77146221  
https://blog.csdn.net/you__are_my_sunshine/article/details/83150722  
https://blog.csdn.net/qq_21430549/article/details/53365915  
https://blog.csdn.net/quwei3930921/article/details/78820991  

ndk 版本  
https://developer.android.google.cn/ndk/downloads/index.html  
https://developer.android.google.cn/ndk/downloads/older_releases.html  
https://dl.google.com/android/repository/android-ndk-r13-linux-x86_64.zip  
https://dl.google.com/android/repository/android-ndk-r13b-linux-x86_64.zip  
https://dl.google.com/android/repository/android-ndk-r13b-windows-x86_64.zip  
