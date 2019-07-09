
1.. 修改 configure  
```
# 老的配置 
# SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
# LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
# SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
# SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'

# 新的配置  
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'

```
2.. 安装 gcc  yaSm
```
sudo apt-get install build-essential
sudo apt-get install yasm  
```
3.. 修改原文  
```
ffmpeg/libavcodec/aaccoder.c  
搜索 B0 改称 b0  
```
4.. 运行脚本  
[alex_ffmpeg.sh](ffmpeg/alex_ffmpeg.md)  
```
./configure  
./alex_ffmpeg.sh 
```
### 参考  
https://ffmpeg.org/download.html  
https://github.com/leixiaohua1020/simplest_ffmpeg_mobile  
https://blog.csdn.net/chenxiaoping1993/article/details/80306928  
https://blog.csdn.net/bobcat_kay/article/details/80889398  
https://cloud.tencent.com/developer/article/1453110  

CMakeLists.text  
https://blog.csdn.net/likuan0214/article/details/78417774  
https://blog.csdn.net/qq_28478281/article/details/87283056  
https://blog.csdn.net/qq_38261174/article/details/83273409  

