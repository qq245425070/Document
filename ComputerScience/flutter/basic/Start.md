####  开始  

安装 SDK  
https://flutter.io/sdk-archive/#windows  
环境变量  
https://flutter.io/setup-windows/#update-your-path  

step 01  
```
cd /Users/alex/WorkSpace/sourceTree  
git clone -b beta https://github.com/flutter/flutter.git  
```

step 02  
```
打开 D:\Program Files (x86)\Git\git-bash.exe  

export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
export PATH=D:\Flutter\flutter\bin:$PATH  

新建环境变量  
FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
PUB_HOSTED_URL=https://pub.flutter-io.cn


#  mac  
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
export FLUTTER_HOME=/Users/alex/WorkSpace/sourceTree/flutterSDK
export PATH=$PATH:$FLUTTER_HOME/bin

#  windows 
FLUTTER_HOME=D:\Flutter\sdk;  
PATH=%FLUTTER_HOME\bin;
PATH=D:\Flutter\sdk\bin;

# ubuntu  
sudo gedit /etc/profile      
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
export FLUTTER_HOME=/home/alex/Flutter/flutter-v1.2.1
export PATH=$PATH:$FLUTTER_HOME/bin

刷新环境变量  
source /etc/profile  
```
step 03  
flutter doctor  

step 04  
flutter upgrade    

step 05  
flutter doctor --android-licenses  

step 06  
https://brew.sh/index_zh-cn  
https://www.dartlang.org/tools/sdk  
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install wget
brew tap dart-lang/dart
brew install dart
brew upgrade dart
```

dart sdk location
```
/Users/alex/WorkSpace/sourceTree/flutterSDK/bin/cache/dart-sdk
```

❀ Flutter 运行 一直Initializing gradle...  
找到  D:\Flutter\flutter\packages\flutter_tools\gradle\flutter.gradle  
```
buildscript {
    repositories {
        mavenLocal()
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/central/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter' }
        jcenter() { url 'http://jcenter.bintray.com/' }
        maven { url "https://jitpack.io" }
        maven { url "https://maven.google.com" }
        google()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.2.1'
    }
}
```
修改 gradle-wrapper.properties  
修改 local.properties  



### 参考  
https://blog.csdn.net/jifashihan/article/details/80675267  
