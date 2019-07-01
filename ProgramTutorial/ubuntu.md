常用命令  
ubuntu sudo root 失败  
```
//  先更新密码  
sudo passwd
//  再获取root权限
su root  
//  打开文件夹  
nautilus .
//  打开, 更新配置文件  
sudo gedit /etc/profile  
source /etc/profile
```


### 环境变量
```
//  sdk ndk  
export ANDROID_NDK=/home/alex/Android/Sdk/android-ndk-r14b
export PATH=$ANDROID_NDK:$PATH
export ANDROID_SDK=/home/alex/Android/Sdk/
export PATH=${ANDROID_SDK}/platform-tools:${PATH}
export PATH=${ANDROID_SDK}/tools:${PATH}

//  gradle 
export GRADLE_HOME=/home/alex/Android/android-studio/gradle/gradle-5.1.1
export PATH=$GRADLE_HOME/bin:$PATH  

// java
export JAVA_HOME=/home/alex/Application/jdk1.8.0_212
export JRE_HOME=${JAVA_HOME}/jre   
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib    
export PATH=${JAVA_HOME}/bin:$PATH 
```
### 新建快捷方式    
新建 studio.desktop  
```
[Desktop Entry]
Version=1.0
Type=Application
Name=Android Studio
Exec="/home/alex/Android/android-studio/bin/studio.sh" %f
Icon=/home/alex/Android/android-studio/bin/studio.png
Categories=Development;IDE;
Terminal=false
StartupNotify=true
StartupWMClass=jetbrains-android-studio
Name[zh_CN]=studio
```
將新建的studio.desktop, 放到 
文件夾下  
cd /home/alex/.local/share/applications 
nautilus .


或者  
文件夾下  
cd /usr/share/applications    
nautilus .  
### 推荐软件  
文件管理器 PCManFM  


