robust 框架有两个插件, robust 负责生成钩子函数, auto-patch-plugin 负责生成 patch 包;  
在编译过程中, robust 会给所有的类, 添加属性 ChangeQuickRedirect changeQuickRedirect;  
每一个函数在调用之前, 都会判断 changeQuickRedirect 对象是否为空, 并且当前方法被加上 @Modify 注解;  

class 访问权限问题;  
super 方法调用的问题;  
```
gradlew clean assembleRelease --stacktrace --no-daemon  
gradle clean assembleRelease --stacktrace --no-daemon  
./gradlew clean assembleRelease --stacktrace --no-daemon  
```

### 参考  
key word  robust 源码分析  
http://w4lle.com/2017/03/31/robust-0/  



https://www.zhihu.com/lives/772803407386800128  

用法  
https://www.jianshu.com/p/d51435895b79  
https://github.com/GaoXiaoduo/gxd-robust  
https://blog.csdn.net/ljw124213/article/details/73844811  
https://blog.csdn.net/junhuahouse/article/details/72465893  

废物  
https://www.cnblogs.com/yrstudy/p/8977315.html  
http://www.520monkey.com/archives/934  
https://blog.csdn.net/jiangwei0910410003/article/details/53705040  

