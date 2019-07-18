[更多命令行](/Linux/linux_cmd.md)  
[应用程序分类-分文件夹](ubuntu/app_dash_board.md)  
常用命令  
ubuntu sudo root 失败  
```
//  先更新密码  
sudo passwd
//  再获取root权限
su root  

//  打开, 更新配置文件  
sudo gedit /etc/profile  
source /etc/profile  
```


### 环境变量  
```
sudo gedit /etc/profile    

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
Icon=/home/alex/Android/android-studio/bin/studio.png
Exec="/home/alex/Android/android-studio/bin/studio.sh" %f
Comment=Android Studio
Categories=Development;IDE;
Terminal=false
StartupWMClass=jetbrains-studio
```

```
將新建的studio.desktop, 放到 
文件夾下
nautilus  /home/alex/.local/share/applications  

cd /home/alex/.local/share/applications 
nautilus .

或者  文件夾下  
nautilus /usr/share/applications

cd /usr/share/applications    
nautilus .  
  
```
配置启动的快捷键  
例如深度截图  
```
nautilus  /usr/share/applications   
找到神图截图的图标, 右键属性, 找到命令 [deepin-screenshot --icon]  
找到设置-设备-键盘-键盘快捷键-自定义:  
名称:  看着写  
命令:  deepin-screenshot --icon  
快捷键: 按下 ctrl + shift + X;  

```
### 修改默认配置  
❀ 按键映射      
```
sudo gedit /usr/share/X11/xkb/symbols/pc  
// 左 control 与 左 window  
key <LCTL> { [ Control_L ] };
key <LWIN> { [ Super_L ] };
//  修改为 
key <LCTL> { [ Super_L ] };
key <LWIN> { [ Control_L ] };
```
❀ 单击图标事件    
```
# 改为点击预览窗口    
gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'previews'
# 改为点击最小化窗口
gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'minimize'
```
intellij  好看的字体是 ubuntu light   

5 gsettings set org.gnome.desktop.default-applications.terminal exec-arg "-x"
❀ 时间显示  
```
sudo apt-get install dconf-tools  
dconf-editor 
org.gnome.desktop.interface.clock-show-date 
```  
❀ 更换主题  
```
sudo apt install gnome-tweak-tool
sudo apt install gnome-shell-extensions
//  好看的主题  
https://github.com/vinceliuice/Mojave-gtk-theme  
sudo apt-get install gtk2-engines-murrine gtk2-engines-pixbuf  
下载下来, 解压缩, 运行 /install.sh  

主题位置  /usr/share/themes/  
icons 位置  /usr/share/icons/  
桌面shell, 状态栏 位置  /usr/share/themes/  


参考  
https://blog.csdn.net/lishanleilixin/article/details/80453565  
```
❀ 安装rpm软件  
```
## alien默认没有安装，所以首先要安装它
sudo apt-get install alien 
##将rpm转换为deb,完成后会生成一个xxxx.deb
sudo alien xxxx.rpm 
##这样xxxx软件就可以安装完成了
sudo dpkg -i xxxx.deb 


视频播放器VLC
sudo apt install vlc  

安装Git  
sudo apt install git  
```
❀ 修改 terminal 字体  
```
sudo apt-get install terminator  
sudo apt install dconf-editor  
好看的字体是 Droid Sans Mono Regular  
```

## 推荐软件  
文件管理器      PCManFM  
压缩软件          ark  Archiving Tool  
压缩软件          P7Zip - Desktop      
画图软件          Krita  

