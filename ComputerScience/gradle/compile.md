❀ setting.gradle  
```
include ':app'  
include ':libraries:someProject'  

if (isLibraryAnnotationSource.toBoolean()) {
    include ':library:apt_annotation'
}
```


❀ 声明依赖项  
```
implementation fileTree(dir: 'libs', include: ['*.jar'])  
implementation 'com.android.support:appcompat-v7:25.0.0'    
implementation project(':YibaAnalytics')    
implementation project(':library:YibaAnalytics')    
implementation files('libs/YibaAnalytics5.jar')    

debugImplementation project(path: ':library', configuration: 'debug')  
releaseImplementation project(path: ':library', configuration: 'release')    
```
api  自己用, 也暴露给 别人用  
implementation  仅仅自己用  
provided 的意思是提供编译支持, 但是不会写入apk;  
androidTestImplementation:  test application
debugImplementation:  debug Build Type  
releaseImplementation:  release Build Type  
runtime: 运行时所需要的依赖;默认情况下, 包含了编译时期的依赖  
testImplementation: 编译测试代码时所需要的依赖;默认情况下, 包含了编译时产生的类文件, 以及编译时期所需要的依赖  
testRuntime: 测试运行时期的依赖;默认情况下, 包含了上面三个时期的依赖  
branchOneImplementation 'com.android.support:appcompat-v7:22.2.0'  //  只为branchOne添加这个依赖  
