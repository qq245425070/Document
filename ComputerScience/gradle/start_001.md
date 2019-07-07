假设我们的插件的 module 叫 plugin_assistant  
开始自定义插件的步骤:  
1.. 新建一个 Android Project;  
2.. 新建一个 Android Module, 类型选择Android Library;  
3.. 修改 build.gradle 的内容, 删除其余无关文件;  
修改模块的 gradle  
```
apply plugin: 'groovy'
apply plugin: 'maven'

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation gradleApi()
    implementation localGroovy()
}

repositories {
    mavenCentral()
}
```
4.. 新建 resources 目录, 和 groovy 目录同级;  
5.. 新建 xxx.properties 文件, 其路径如下:  
```
resources/META-INF/gradle-plugins/org.alex.plugin.assistant.properties  
```
这个 properties 的文件, 他的文件名很重要,  如果我们用到这个插件的话, apply plugin: 'org.alex.plugin.assistant'    
没错,  properties 的名字,  和 apply plugin 的 索引值, 是一样的;  
其内容如下:  
```
// 全类名  
implementation-class=org.alex.plugin.assistant.AssistantPlugin
```
6.. 新建文件 AssistantPlugin.groovy;  
7.. 写代码  
```
package org.alex.plugin.assistant

import org.gradle.api.Plugin
import org.gradle.api.Project;

public class AssistantPlugin implements Plugin<Project> {

    @Override
    void apply(Project project) {

    }
}
```
❀ 再写一个插件  
假设我们的插件的 module 叫 plugin_aspectj  
那么 plugin_aspectj 的 build.gradle   
```
apply plugin: 'groovy'
apply plugin: 'maven'
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile gradleApi()
    compile localGroovy()
    //  下面两个 是扩展需要的  
    compile 'org.aspectj:aspectjtools:1.8.13'
    compile 'org.aspectj:aspectjrt:1.8.13'
    compile 'com.android.tools.build:gradle:3.0.1'
}
repositories {
    mavenCentral()
}
sourceSets {
    //  注意,  groovy 的代码, 必须写在 groovy , 这个主包内,  写在 java, 这个主包内, 就不行,  暂时 这么理解  
    main {
        groovy {
            srcDir 'src/main/groovy'
        }

        resources {
            srcDir 'src/main/resources'
        }
    }
}
uploadArchives {
    repositories {
        mavenDeployer {
            //设置插件的GAV参数, 和 classpath 'org.alex.plugin:aspectj:1.0.0'  一致的
            pom.groupId = 'org.alex.plugin'
            pom.artifactId = 'aspectj'
            pom.version = '1.0.0'
            //文件发布到下面目录,  minePluginUri = file:///Users/alex/WorkSpace/Gradle/MinePlugin/repo/  
            repository(url: uri(minePluginUri))
        }
    }
}

```  


properties    
resources/META-INF/gradle-plugins/org.alex.plugin.aspectj.properties  
这个 properties 的文件, 他的文件名很重要,  因为如果 我们用到  这个插件的话, apply plugin: 'org.alex.plugin.aspectj'    
没错,  properties 的名字,  和 apply plugin 的 索引值, 是一样的;  
记得先执行 uploadArchives,  再同步 和 引用;  

