### aspectj  
[aspect for android](/Java/framework/aspectj/android_config.md)  
[基础知识](aspectj/basic.md)  


实例介绍 1   
```
@Before("execution(* com.alex.andfun.*.*Activity.onCreate(..)) ")  
```
com.alex.andfun 包及子包内, 所有的以 Activity 为后缀的类, 的onCreate 方法;   


```
execution(* com.alex.andfun.apt.AptMainActivity.onCreate(..))
```
说明  匹配com.alex.andfun.apt.AptMainActivity, 这个类的, onCreate方法;  


```
@Before("within(com.alex.andfun.apt.model.AopJustInterface+) && execution(* com.alex.andfun..*.onCreate(..)) ")  
@Before("within(android.app.Activity+) && execution(* com.alex.andfun..*.onCreate(..)) ")  
```
匹配  
类A 是AopJustInterface 的子类, 或者实现类;  
类A 在com.alex.andfun  包及子包内;  
类A 的onCreate 方法;    



execution: 用于匹配方法执行的连接点;  
within: 用于匹配指定类型内的方法执行;

### 依赖库的问题  
如果 app  依赖 tools  
那么需要在 app.gradle 和 tools.gradle 都配置上   
```
apply plugin: 'org.alex.plugin.aspectj'
```
   
  

### 参考  
https://github.com/HujiangTechnology/gradle_plugin_android_aspectjx  
http://www.eclipse.org/aspectj/doc/released/runtime-api/index.html  
https://eclipse.org/aspectj/doc/released/progguide/index.html  
https://blog.csdn.net/innost/article/details/49387395  
https://fernandocejas.com/2014/08/03/aspect-oriented-programming-in-android/  
http://jinnianshilongnian.iteye.com/blog/1415606  
http://blog.csdn.net/zhengchao1991/article/details/53391244  
http://blog.csdn.net/qwe6112071/article/details/50951720    
https://juejin.im/post/5c57b2d5e51d457ffd56ffbb  
http://blog.51cto.com/lavasoft/172292  
http://www.cnblogs.com/yudy/archive/2012/03/22/2411175.html  
https://www.ibm.com/developerworks/cn/java/j-lo-springaopcglib  
https://juejin.im/post/5c01533de51d451b80257752  

原理  
apt, javapoet, hugo, aspectJ, aspectJX, javassist, savior, asm, ASMDEX,   

