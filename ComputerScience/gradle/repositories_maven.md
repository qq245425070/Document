```

buildscript {

    repositories {
        mavenLocal()
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/central/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter' }
        jcenter() { url 'http://jcenter.bintray.com/' }
        maven { url "https://jitpack.io" }
        maven { url "https://maven.google.com" }
        google()
        mavenCentral()
        maven { url minePluginUri }
    }
    
}

allprojects {
    repositories {
        mavenLocal()
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/central/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter' }
        jcenter() { url 'http://jcenter.bintray.com/' }
        maven { url "https://jitpack.io" }
        maven { url "https://maven.google.com" }
        google()
        mavenCentral()
        maven { url minePluginUri }

        /*设置 aar 路径 */
        flatDir {
            dirs '../build_jar_aar'
        }
    }
    
}

```

### 添加#aar     
在全局的 gradle  
```
allprojects {
    repositories {
        jcenter()

        flatDir {
            dirs '../build_jar_aar'
        }
    }
}
```
在模块的gradle  
```
dependencies {
    compile(name:'YibaAnalytics-release', ext:'aar')
}
```  