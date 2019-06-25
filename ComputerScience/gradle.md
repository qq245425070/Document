[Transform](gradle/Transform.md)  
插件 version 与 gradle version 的对应关系;  
minSdkVersion-targetSdkVersion-compileSdkVersion 三者关系;  
[链接](gradle/gradle_plugin_version.md)  

[常见问题](gradle/common_bug.md)  

声明依赖项;  解决依赖冲突;  
compile;  implementation;  debugImplementation;  
releaseImplementation;  androidTestImplementation;  
repositories;  配置maven库的地址;  添加 aar 包;  配置 aar 包;  
修改 build产生的临时文件的位置;  
[链接](gradle/compile.md)  

加快编译速度, 优化apk大小;  
配置签名工具;  
applicationId 添加后缀;  BuildConfig 添加字段;  
externalNativeBuild; ndk 配置;  so 库配置;  splits;  resConfigs;  
[链接](gradle/android_buildTypes.md)  

修改apk名字;  productFlavors 构建不同产品;  渠道包;  
[链接](gradle/productFlavors.md)  

[自定义lint](lint/lint.md)  
[开始自定义plugin](gradle/start_001.md)  
[自定义 localMaven](gradle/localMaven.md)   

自定义 localMaven 添加Java 文档;  
[链接](gradle/localMavenJavaDoc.md)   

[lintOptions](gradle/lintOptions.md)  

sourceSets 修改源集;  
java 代码的目录结构;  
res 代码的对应结构;    
[链接](gradle/sourceSets.md)  

在一个窗口里面打开多个 project;  
不是打开多个 module;  
[链接](gradle/one_window_more_project.md)  

### gradle脚本
外部函数的声明与引用;  
[链接](gradle/def_fun_outter.md)  

gradle task 详解;  gradle 任务详解;  makeJar;  打 jar 包;  
[链接](gradle/task_sample_01.md)  

gradle 脚本的基本使用;  
[链接](gradle/basic_method.md)  
### 延伸方法  
```
aaptOptions { }  
adbOptions { }  
buildTypes { }  
compileOptions { }  
dataBinding { }  
defaultConfig { }  
dexOptions { }   
externalNativeBuild { }  
jacoco { }  
lintOptions { }  
packagingOptions { }  
productFlavors { }  
signingConfigs { }  
sourceSets { }  
splits { }  
testOptions { }  
```
compileOptions     
```
compileOptions {  
    sourceCompatibility JavaVersion.VERSION_1_7  
    targetCompatibility JavaVersion.VERSION_1_7  
}
```

  
### Gradle常见命令  
通用 gradle, 需要配置环境变量;  windows  gradlew;  mac  ./gradlew;  
gradle app:assembleDebug  
//构建  
gradlew app:clean    //移除所有的编译输出文件, 比如apk    

gradlew app:build   //构建 app module , 构建任务, 相当于同时执行了check任务和assemble任务  

//检测  
gradlew app:check   //执行lint检测编译;  

//打包  
gradlew app:assemble //可以编译出release包和debug包, 可以使用gradlew assembleRelease或者gradlew assembleDebug来单独编译一种包   

gradlew app:assembleRelease  //app module 打 release 包   

gradlew app:assembleDebug  //app module 打 debug 包  

//安装, 卸载  
gradlew app:installDebug  //安装 app 的 debug 包到手机上  

gradlew app:uninstallDebug  //卸载手机上 app 的 debug 包   

gradlew app:uninstallRelease  //卸载手机上 app 的 release 包   

gradlew app:uninstallAll  //卸载手机上所有 app 的包   

gradlew assembleWandoujiaRelease  //豌豆荚 release 包  

gradlew assembleWandoujiaDebug //豌豆荚 debug 包  

gradle -q projects  
收集项目信息  
```
Root project 'Sample'
+--- Project ':app'
+--- Project ':library_middleware'
+--- Project ':library_module'
\--- Project ':library_platform'
```

### groovy#api  
```
private String getAddress(Project project) {
    //  首先判断 project 中有没有 myAddress, 一般在 gradle.properties 文件中定义属性, 没有则使用配置的属性值  
    project.hasProperty('myAddress') ? project.myAddress : project.extensions.findByName(EXTENSION_NAME).myAddress
}

/*获取扩展属性的信息*/
private static List<String> packageList(Project project) {
    def block = project.extensions.findByName(AssistantBlock.NAME)
    if (block == null) {
        return new ArrayList<String>()
    }
    def packageList = block.properties.get(AssistantBlock.ARGS_PACKAGE_LIST)
    return packageList == null ? new ArrayList<String>() : packageList
}
```

#### Transform   
```
//transform的名称
//transformClassesWithMyClassTransformForDebug 运行时的名字
//transformClassesWith + getName() + For + Debug或Release
@Override
public String getName() {
    return "MyClassTransform";
}

//需要处理的数据类型, 有两种枚举类型
//CLASSES和RESOURCES, CLASSES代表处理的java的class文件, RESOURCES代表要处理java的资源
@Override
public Set<QualifiedContent.ContentType> getInputTypes() {
    return TransformManager.CONTENT_CLASS;
}

//    指Transform要操作内容的范围, 官方文档Scope有7种类型：
//    EXTERNAL_LIBRARIES        只有外部库
//    PROJECT                       只有项目内容
//    PROJECT_LOCAL_DEPS            只有项目的本地依赖(本地jar)
//    PROVIDED_ONLY                 只提供本地或远程依赖项
//    SUB_PROJECTS              只有子项目;
//    SUB_PROJECTS_LOCAL_DEPS   只有子项目的本地依赖项(本地jar);
//    TESTED_CODE                   由当前变量(包括依赖项)测试的代码
@Override
public Set<QualifiedContent.Scope> getScopes() {
	return TransformManager.SCOPE_FULL_PROJECT;
}

//指明当前Transform是否支持增量编译
@Override
public boolean isIncremental() {
    return false;
}

//    Transform中的核心方法, 
//    inputs中是传过来的输入流, 其中有两种格式, 一种是jar包格式一种是目录格式;
//    outputProvider 获取到输出目录, 最后将修改的文件复制到输出目录, 这一步必须做不然编译会报错
//  Installation failed since the device possibly has stale dexed jars that dont match the current version(dexopt error)  
//  In order to proceed, you have to unstall the existing application
//  就算卸载, 也还是安装不上, 因为没有完成文件的拷贝;  
@Override
void transform(Context context, Collection<TransformInput> inputs, Collection<TransformInput> referencedInputs, TransformOutputProvider outputProvider, boolean isIncremental) throws IOException, TransformException, InterruptedException {
    LogTrack.w("transform 开始")
    if (inputs == null || inputs.isEmpty()) {
        return
    }
    inputs.each { TransformInput input ->

        input.directoryInputs.each { DirectoryInput directoryInput ->
            if (directoryInput.file.isDirectory()) {
                directoryInput.file.eachFileRecurse { File file ->
                    LogTrack.w(file.name)
                }
            }
            def dest = outputProvider.getContentLocation(directoryInput.name, directoryInput.contentTypes, directoryInput.scopes, Format.DIRECTORY)
            FileUtils.copyDirectory(directoryInput.file, dest)
        }

        input.jarInputs.each { JarInput jarInput ->
            def jarName = jarInput.name
            def md5 = DigestUtils.md5Hex(jarInput.file.getAbsolutePath())
            if (jarName.endsWith(".jar")) {
                jarName = jarName.substring(0, jarName.length() - 4)
            }
            def dest = outputProvider.getContentLocation(jarName + md5, jarInput.contentTypes, jarInput.scopes, Format.JAR)
            FileUtils.copyFile(jarInput.file, dest)
        }
    }
    LogTrack.w("transform 结束")
}
```

### 参考  
https://docs.gradle.org/current/dsl/org.gradle.api.invocation.Gradle.html  
https://dongchuan.gitbooks.io/gradle-user-guide-/content/  
https://chaosleong.gitbooks.io/gradle-for-android/content/index.html  
https://blog.csdn.net/xu_coding/article/details/80819494  
http://google.github.io/android-gradle-dsl/current/  
https://github.com/udacity/ud867  
https://blog.csdn.net/Innost/article/details/48228651  
https://developer.android.google.cn/studio/build/index.html  
https://juejin.im/post/582d606767f3560063320b21  
https://blog.csdn.net/linkuiyao/article/details/78079871
https://github.com/rujews/android-tech-docs/blob/master/new-build-system/user-guide/README.md  
http://gradledoc.qiniudn.com/1.12/userguide/userguide.html  
https://www.jianshu.com/p/e26236943dd6  

layout 分包  
https://www.jianshu.com/p/b27fbec5b87b  

Transform  
https://github.com/JieYuShi/Luffy  

调试  
https://www.colabug.com/2109876.html  
https://www.jianshu.com/p/b4c44e62d652  

插件  
https://github.com/scana/ok-gradle  
https://docs.gradle.org/current/userguide/custom_plugins.html    

不需要看了  
http://www.cnblogs.com/zhaoyanjun/p/7603640.html  
https://lippiouyang.gitbooks.io/gradle-in-action-cn/content/    
https://chaosleong.gitbooks.io/gradle-for-android/content/  
https://avatarqing.gitbooks.io/gradlepluginuserguidechineseverision/content/  
https://flyouting.gitbooks.io/gradle-plugin-user-guide-cn/content/  
http://www.paincker.com/gradle-develop-basics  
http://www.flysnow.org/categories/Android/  
https://www.jianshu.com/p/7b31cc80421d  


