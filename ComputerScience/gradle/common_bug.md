❀ 错误: 编码GBK的不可映射字符  
在当前 module 的 build.gradle 添加  
```
tasks.withType(JavaCompile) {
    options.encoding = 'UTF-8'
}
```
全局的 gradle  
```
allprojects {
    repositories {
        google()
        jcenter()
    }
    tasks.withType(JavaCompile){
        options.encoding = "UTF-8"
    }
}
```

❀ 解决版本冲突  
全局的 build.gradle  
```
allprojects {
    configurations.all {
        resolutionStrategy {
                force "com.android.support:appcompat-v7:25.1.0"
        }
        //  过滤掉 所有的 support
        exclude group: 'com.android.support'
    }
}
```
当前 module.gradle  
```
dependencies {
    androidTestImplementation('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    implementation ('com.alex.tools:log-dev:1.0.11') {
            exclude group: 'com.android.support', module: 'palette-v7'
            exclude group: 'com.android.support', module: 'appcompat-v7'
            exclude group: 'com.android.support', module: 'recyclerview'
            force=true
            //  加上 force = true 即使在有依赖库版本冲突的情况下, 坚持使用被标注的这个依赖库版本  
        }
    implementation ('com.alex.tools:log-dev:1.0.11'){
        exclude module: 'base_model'    
        exclude group:'com.name.group' module:'base_model'
    }
    
}
```

❀ AAPT Error  
Exception while processing task java.io.IOException: Unable to make AAPT link command.  
修改 gradle.properties  
```
android.enableAapt2=false
```
出现  
```
> Failed to process resources, see aapt output above for details.
```
