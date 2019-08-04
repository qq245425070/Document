ClassFileTransformer  
Transform  

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
https://github.com/Leaking/Hunter  
https://github.com/houjinyun/Android-AppLifecycleMgr  
https://www.jianshu.com/p/16ed4d233fd1  
