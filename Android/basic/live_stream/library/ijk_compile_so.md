ubuntu 下编译 ijkPlayer.so 文件, 其实并不需要打开 android studio, 或者讲这过程和androidStudio 没有任何的关系;  
环境变量
```
//  sdk ndk  
export ANDROID_NDK=/home/alex/Android/Sdk/android-ndk-r14b
export PATH=$ANDROID_NDK:$PATH
export ANDROID_SDK=/home/alex/Android/Sdk/
export PATH=${ANDROID_SDK}/platform-tools:${PATH}
export PATH=${ANDROID_SDK}/tools:${PATH}
```
克隆仓库  
```
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-android  
cd ijkplayer-android  
git checkout -B latest k0.8.8  
```
执行编译过程, 要耐心等待, 理论上有一些操作, 会等很久的;  
```
cd  /home/alex/WorkSpace/AndroidStudio/ijkplayer-android  
./init-android.sh  

//  支持Https协议  
./init-android-openssl.sh  

//  编译openssl  
cd /home/alex/WorkSpace/AndroidStudio/ijkplayer-android/android/contrib  
./compile-openssl.sh clean
./compile-openssl.sh all

cd /home/alex/WorkSpace/AndroidStudio/ijkplayer-android/config  
rm module.sh
ln -s module-lite.sh module.sh  

//  编译 ffmpeg  
cd /home/alex/WorkSpace/AndroidStudio/ijkplayer-android/android/contrib  
./compile-ffmpeg.sh clean  
./compile-ffmpeg.sh all  

//  编译ijkplayer  
cd /home/alex/WorkSpace/AndroidStudio/ijkplayer-android/android  
./compile-ijk.sh all  
或者
./compile-ijk.sh armv5|armv7a|x86|arm64-v8a #编译指定版本

//  查看编译出来的 so 文件  
cd /home/alex/WorkSpace/AndroidStudio/ijkplayer-android/android/ijkplayer/ijkplayer-armv7a/src/main/libs/armeabi-v7a  
ls  或者 nautilus . 
```

