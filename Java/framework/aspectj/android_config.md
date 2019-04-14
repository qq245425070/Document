
使用原生aspectj 配置建麻烦，还有不兼容问题，建议使用hugo插件  
### android  aspectJ  配置  

gradle.properties  
aspectj_version = 1.8.13  

全局的 gradle  
/*aspectJ */   
classpath "org.aspectj:aspectjtools:${aspectj_version}"  
classpath "org.aspectj:aspectjweaver:${aspectj_version}"  

模块的 gradle  
//  不能使用 aspectjtools 否则会，一直报  Could not determine java version from '191'.  
//  implementation 'org.aspectj:aspectjtools:1.8.13'  
implementation 'org.aspectj:aspectjrt:1.8.13'  
```
android {

}
kapt {

}
import org.aspectj.bridge.IMessage
import org.aspectj.bridge.MessageHandler
import org.aspectj.tools.ajc.Main
final def log = project.logger
final def variants = project.android.applicationVariants

variants.all { variant ->
    if (!variant.buildType.isDebuggable()) {
        log.debug("Skipping non-debuggable build type '${variant.buildType.name}'.")
        return;
    }

    JavaCompile javaCompile = variant.javaCompile
    javaCompile.doLast {
        String[] args = ["-showWeaveInfo",
                         "-1.5",
                         "-inpath", javaCompile.destinationDir.toString(),
                         "-aspectpath", javaCompile.classpath.asPath,
                         "-d", javaCompile.destinationDir.toString(),
                         "-classpath", javaCompile.classpath.asPath,
                         "-bootclasspath", project.android.bootClasspath.join(File.pathSeparator)]
        log.debug "ajc args: " + Arrays.toString(args)

        MessageHandler handler = new MessageHandler(true);
        new Main().run(args, handler);
        for (IMessage message : handler.getMessages(null, true)) {
            switch (message.getKind()) {
                case IMessage.ABORT:
                case IMessage.ERROR:
                case IMessage.FAIL:
                    log.error message.message, message.thrown
                    break;
                case IMessage.WARNING:
                    log.warn message.message, message.thrown
                    break;
                case IMessage.INFO:
                    log.info message.message, message.thrown
                    break;
                case IMessage.DEBUG:
                    log.debug message.message, message.thrown
                    break;
            }
        }
    }
}

```

### hugo   
https://github.com/JakeWharton/hugo  

全局的gradle  
classpath 'com.jakewharton.hugo:hugo-plugin:1.2.1'  
模块的gradle  
apply plugin: 'com.jakewharton.hugo'  
```
android {

}
kapt {

}
hugo {
  enabled false
}
```

假设插入的方法是 fooA;  目标方法是 funA;  
call 和 execution  
execution  插入的 代码，在目标 方法  内部， 形成  
```
funA(){
    fooA();
    // 实际 方法体  
}
funA(){
    // 实际 方法体  
    fooA();
}
```
call 插入的 代码，在目标 方法  外部， 形成  
```
methodA(){
    fooA();
    funA();  
}
methodA(){
    funA();  
    fooA();
}
```

@Before("execution(* android.app.Activity.onCreate(..))")  
@Before("execution(* android.app.Activity.on*(..))")  
第一个『*』表示返回值，『*』表示返回值为任意类型  
第二个『*』来进行通配，几个『*』没区别。  



###参考  
https://www.jianshu.com/p/5c9f1e8894ec（不看了）  
https://www.jianshu.com/p/f90e04bcb326  
https://www.jianshu.com/p/27b997677149  
https://www.jianshu.com/p/6ccfa7b50f0e   
https://www.jianshu.com/p/dca3e2c8608a  
https://github.com/uPhyca/gradle-android-aspectj-plugin  
http://blog.csdn.net/woshimalingyi/article/details/51519851  
http://blog.csdn.net/weelyy/article/details/78987087  
http://blog.csdn.net/qq_25943493/article/details/52524573    
http://blog.csdn.net/innost/article/details/49387395  
https://www.jianshu.com/p/430f9ea1e80f  
http://jinnianshilongnian.iteye.com/blog/1415606  
http://blog.csdn.net/qq_32719003/article/details/71404428  
https://juejin.im/post/5c57b2d5e51d457ffd56ffbb  
https://yq.aliyun.com/articles/7104  
https://github.com/JakeWharton/hugo  
https://github.com/HujiangTechnology/gradle_plugin_android_aspectjx  






