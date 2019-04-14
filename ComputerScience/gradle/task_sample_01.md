gradle taskA   //  会有多余的信息输出;  
gradle -q taskA   //  只有脚本内部的输出;  
```
//  只要 sync 一下, taskA 会立即执行;  
task taskA {  logV "helloA"  }

//  需要手动调用 gradle taskA, taskA 才会执行;  
task taskA << {  logV "helloA"  }

//  手动执行, 左边有三角形的 run 按钮, 省去敲命令行了;   
task taskA() << {
    //  描述信息  
    description '任务的描述信息'  
    logV "helloA"  
}
```

### dependsOn  
C dependsOn B;  
gradle C: 先执行 B, 再执行 C;  
如果在执行 B 的过程中, 出现了异常, 那么 B 会停下来, 当然更不会继续执行 C了;  
```
task taskA << {
    logV "helloA"
}

task taskB << {
    logV "helloB"
}

task taskC(dependsOn: taskB) << {
    logV "helloC"
}

task taskD {
    dependsOn = ["taskA", "taskB"]
}

task taskD(dependsOn: [taskA, taskB]) << {
    logV "helloD"
}

//  或者在外部声明依赖关系  
taskX.dependsOn taskY  

// gradle taskC  输出:  helloB  helloC  
```

❀ 过滤掉其中一个 task;  
假设 taskD 依赖 taskA, taskB, taskC;  
gradle taskD -x taskA;  

❀ 通过闭包加入依赖  
```
task taskX << {
    println 'taskX'
}

taskX.dependsOn {
    tasks.findAll { task -> task.name.startsWith('lib') }
}

task lib1 << {
    println 'lib1'
}

task lib2 << {
    println 'lib2'
}

task notALib << {
    println 'notALib'
}
```
gradle -q taskX 输出: lib1  lib2  taskX  

❀ 得到一个 task  
```
task taskE()<<{
    logV tasks.taskC
    logV tasks['taskC']
    logV tasks.getByPath("taskC")
    logV tasks.getByPath(":project:hello")
}
```
❀ 跨模块依赖  
在 D:\WorkSpace\Android\Gradle\Sample\library_platform\build.gradle   
```
task taskX << {
    logV "项目的绝对路径   ${project.projectDir}"
}
```
在 D:\WorkSpace\Android\Gradle\Sample\library_module\build.gradle  
```
task taskY(dependsOn: ':library_platform:taskX') << {
    logV "项目的绝对路径   ${project.projectDir}"
}
```
执行 gradle -q taskY  


❀ 排序关系  
注意: "B.mustRunAfter(A)" 或者 "B.shouldRunAfter(A)" 并不影响任何任务间的执行依赖(dependsOn);  
```
task taskX << {
    println 'taskX'
}
task taskY << {
    println 'taskY'
}
taskY.mustRunAfter taskX  
taskY.shouldRunAfter taskX  
```

❀ 替换 tasks  
有时候你想要替换一个任务, 举个例子, 如果你想要互换一个通过 java 插件定义的任务和一个自定义的不同类型的任务;  
```
task copy(type: Copy)

task copy(overwrite: true) << {
    println('I am the new one.')
}

```
这种方式将用你自己定义的任务替换一个 Copy 类型的任务, 因为它使用了同样的名字;   
但你定义一个新的任务时, 你必须设置 overwrite 属性为 true, 否则的话 Gradle 会抛出一个异常, task with that name already exists;  

❀ 跳过 tasks  

使用判断条件 onlyIf   
```
task hello << {
    println 'hello world'
}

hello.onlyIf { !project.hasProperty('skipHello') }

```

抛异常跳过  
如果想要跳过一个任务的逻辑并不能被判断条件通过表达式表达出来, 你可以使用 StopExecutionException.   
如果这个异常是被一个任务要执行的动作抛出的, 这个动作之后的执行以及所有紧跟它的动作都会被跳过. 构建将会继续执行下一个任务.  
```
task compile << {
    println 'We are doing the compile.'
}

compile.doFirst {
    // Here you would put arbitrary conditions in real life.
    // But this is used in an integration test so we want defined behavior.
    if (true) { throw new StopExecutionException() }
}
task myTask(dependsOn: 'compile') << {
   println 'I am not affected'
}
```

注销和激活任务  
```
task disableMe << {
    println 'This should not be printed if the task is disabled.'
}
disableMe.enabled = false
```

❀ finalizedBy  
类似于 finally, taskX.finalizedBy taskY, 无论 taskX 是否出现异常, 最终都会触发执行 taskY;  

### makeJar  
需求，根据class文件，生成jar包  
假设，class文件在：  build/intermediates/classes/basic/release  
在 module 的 gradle文件  
```
task makeJar(type: Jar) {
    baseName 'AndFun_App'
    from('build/intermediates/classes/basic/release')
    into( 'build/libs')
    exclude('android/')
    exclude('**/BuildConfig.class')
    exclude('**/BuildConfig\$*.class')
    exclude('**/R.class')
    exclude('**/R\$*.class')
    include('**/*.class')
}
makeJar.dependsOn(build)
```

```
task makeJar(type: Jar) {
    baseName 'alex_tools'
    version = "1.0.0"
    from('build/intermediates/classes/debug/')
//    into( 'build/libs')
    exclude('android/')
    exclude('**/BuildConfig.class')
    exclude('**/BuildConfig\$*.class')
    exclude('**/R.class')
    exclude('**/R\$*.class')
    exclude('com/**')
    exclude('io/**')
    include('/org/alex/util/**/*.class')
    include('/org/alex/okhttp/**/*.class')
}
makeJar.dependsOn(build)
```

### task.copy   
```
task copyDocs(type: Copy) {
    logV "该脚本所在目录的绝对路径   ${project.projectDir}"  //  项目的绝对路径   D:/WorkSpace/Android/Gradle/Sample/library_platform  
    from 'src/main/fileA'
    into 'build/target/doc'
}

例如当前的路径是 D:/WorkSpace/Android/Gradle/Sample  
如果在sample目录下面, 存在一个 fileA 的文件,  
其相对路径是 src/main/fileA  
其据对路径是 D:/WorkSpace/Android/Gradle/Sample/src/main/fileA  
那么会把它拷贝到 相对路径 build/target/doc 目录下  
绝对路径  D:/WorkSpace/Android/Gradle/Sample/build/target/doc  
```
其他参数  
```
task copyDocs(type: Copy) {
    logV "项目的绝对路径   ${project.projectDir}"  //  项目的绝对路径   D:/WorkSpace/Android/Gradle/Sample  
    from 'src/main/fileA'
    //  包含 
    include '*.data'  
    include '**/*.properties'
    include '**/*.xml'
    include('**/*.txt', '**/*.xml', '**/*.properties')  
    
    //  过滤掉  
    exclude '**/*.properties', '**/*.xml'  
    exclude { details -> details.file.name.endsWith('.html') &&
                             details.file.text.contains('staging') 
    }

    //  重命名  
    rename 'EN_US_(.*)', '$1'  
    rename { String fileName ->
            fileName.replace('-staging-', '')
    }
    
    //  使用正则表达式映射文件名
    rename '(.+)-staging-(.+)', '$1$2'
    rename(/(.+)-staging-(.+)/, '$1$2')
    
        
    //  包含空的目录  
    includeEmptyDirs = false  
    
    into 'build/target/doc'
}
```


### 参考   
https://github.com/rujews/android-gradle-book-code/tree/master/chapter04  
http://wiki.jikexueyuan.com/project/GradleUserGuide-Wiki/more_about_tasks/defining_tasks.html  

task copy 参数详解  
https://docs.gradle.org/current/dsl/org.gradle.api.tasks.Copy.html  



