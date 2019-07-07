### app/build.gradle  
```
android {
    defaultConfig {
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
            path "src/main/cpp/CMakeLists.txt"
            version "3.10.2"
        }
    }

    buildTypes {
        
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
}

```
### MainActivity  
```
class MainActivity : AppCompatActivity() {
    companion object {
        init {
            //  #  引用 libalex-hello.so 文件
            System.loadLibrary("alex-jni-fun")
        }
    }
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        textView.text = stringFromJNI()
    }
    external fun stringFromJNI(): String
}
```
### CMakeLists.txt  
app/src/main/cpp/CMakeLists.txt  
```
cmake_minimum_required(VERSION 3.4.1)

add_library(
        #  生成 libalex-jni-fun.so 文件
        alex-jni-fun
        SHARED
        HelloFun.cpp
        LogTrack.cpp)


find_library(
        log-lib
        log)

target_link_libraries(
        #  引用 libalex-jni-fun.so 文件
        alex-jni-fun
        ${log-lib})

```

### HelloFun.cpp  
app/src/main/cpp/HelloFun.cpp  
```
#include <jni.h>
#include <string>
#include<android/log.h>
#include "LogTrack.cpp"

extern "C" JNIEXPORT jstring JNICALL
Java_com_alex_cmakefun_demo_MainActivity_stringFromJNI(
        JNIEnv *env,
        jobject) {
    std::string hello = "Hello from C++";
    LOGV("hello, message=%s\n", hello.c_str());
    return env->NewStringUTF(hello.c_str());
}
```
