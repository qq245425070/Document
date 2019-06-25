1.. setting.gradle  
```
def gradleLibraryDir = "gradle_library"
apply from: "${gradleLibraryDir}/includeSource.gradle"
include ':app'

def overOneTrue = hasOverOneTrue()
def libProjectName = getLibProjectName()
if (overOneTrue) {
    // 只要有一个源码依赖, 就展示出来外部依赖 project
    include ":${libProjectName}"
    project(":${libProjectName}").projectDir = new File("../${libProjectName}")
}
if (isLibPlatformSource.toBoolean()) {
    include ":${libProjectName}:library_platform"
}
```
2.. Sample/build.gradle  
```
def libProjectName = getLibProjectName()
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    if (isLibPlatformSource.toBoolean()) {
        implementation project(":${libProjectName}:library_platform")
    } else {
        implementation libs.gsl_platform
    }
}
```
3.. Sample/gradle.properties  
```
# platform 源码
isLibPlatformSource=true
isLibPlayerSource=true2
isLibPlayerSkinSource=true2
isLibLiveIjkSource=true2
```
4.. includeSource.gradle  
```
private static Properties loadProperties(File propertiesFile) {
    Properties properties = new Properties()
    if (null == propertiesFile || !propertiesFile.exists()) {
        return properties
    }
    InputStream inputStream = propertiesFile.newDataInputStream()
    properties.load(inputStream)
    return properties
}

static boolean hasOverOneTrue() {
    def properties = loadProperties(new File("gradle.properties"))
    def isSource = false
    properties.propertyNames().each {
        if (it.toString().startsWith("isLib")) {
            isSource |= properties.getProperty(it).toBoolean()
        }
    }
    return isSource
}

static String getLibProjectName() {
    return "AndFun"
}
/*函数在这里声明， 其他地方 直接调用*/
ext {
    hasOverOneTrue = this.&hasOverOneTrue
    getLibProjectName = this.&getLibProjectName
}
```
### 最后的现象  
```gradle
AndFun  
    |library_platform
Sample
    |app  
    |modlue_zxing
    |module_toast    
    |setting.gradle  
    |gradle.properties
```