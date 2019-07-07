### 修改apk名字  
在app的module里的build.gradle文件中, 在android { ...}里面加上这样一段代码, 即可修改生成的apk的文件名;  

```
applicationVariants.all { variant ->
    variant.outputs.all { output ->
        def outputFile = output.outputFile
        if (outputFile != null && outputFile.name.endsWith('.apk')) {
            //  删除  之前打包的 apk
            def file = new File(outputFile.getParentFile().getParentFile().getParentFile().getAbsolutePath())
            file.deleteDir()
            //生成新的apk  AndFun_basic_debug_1.0_2018_0311_1203.apk
            def fileName = "AndFun_${variant.flavorName}_${variant.buildType.name}_${defaultConfig.versionName}_${buildTime()}.apk"
            outputFileName = fileName
        }
    }
}
//  依赖函数
static def buildTime() {
    def date = new Date()
    def formattedDate = date.format('yyyy_MMdd_HHmm')
    return formattedDate
}  
```
### 修改 flavor  
每一个不同的 productFlavors 都可以覆盖 applicationId  
如果使用 gradle 3.0以上, 那么 flavor 必须有一个 dimension, 他是用来做 flavor 分组用的;  
```
android {
    ...
    // Specifies the flavor dimensions you want to use. The order in which you
    // list each dimension determines its priority, from highest to lowest,
    // when Gradle merges variant sources and configurations. You must assign
    // each product flavor you configure to one of the flavor dimensions.
    flavorDimensions 'api', 'version'

    productFlavors {
      demo {
        // Assigns this product flavor to the 'version' flavor dimension.
        dimension 'version'
        packageName "com.example.flavor1"
        versionCode 20
        ...
    }

      full {
        dimension 'version'
        applicationId "com.alex.myapplication"
        packageName "com.example.flavor2"
        applicationIdSuffix ".demo"
        versionNameSuffix "-demo"
        minSdkVersion 14
        buildConfigField "String", "FLAVORE_NAME", '"common"'
        buildConfigField "int", "me_flavor", "2"
        ...
      }

      minApi24 {
        // Assigns this flavor to the 'api' dimension.
        dimension 'api'
        minSdkVersion '24'
        versionNameSuffix "-minApi24"
        ...
      }

      minApi21 {
        dimension "api"
        minSdkVersion '21'
        versionNameSuffix "-minApi21"
        ...
      }
   }
   
   productFlavors.all { flavor ->
       def name = flavor.name
       def nameFormat = quotation(name)
       if ("common".equalsIgnoreCase(name)) {
           flavor.versionNameSuffix = ""
       } else {
           flavor.versionNameSuffix = "-$name"
       }
       buildConfigField "String", "FLAVORE_NAME", nameFormat
       manifestPlaceholders.put("UMENG_CHANNEL", name)
   }
   
}
```

flavor 有哪些属性?  
```
productFlavors {
    //  普通版本
    common {
        logV productFlavors[0]
        dimension 'business'
        buildConfigField "String", "FLAVORE_NAME", '"common"'
        //  resConfigs "en", "xxhdpi"
    }
}


ProductFlavor_Decorated{
    name=common, 
    dimension=null, 
    minSdkVersion=null, 
    targetSdkVersion=null, 
    renderscriptTargetApi=null, 
    renderscriptSupportModeEnabled=null, 
    renderscriptSupportModeBlasEnabled=null, 
    renderscriptNdkModeEnabled=null, 
    versionCode=null, 
    versionName=null, 
    applicationId=null, 
    testApplicationId=null, 
    testInstrumentationRunner=null, 
    testInstrumentationRunnerArguments={}, 
    testHandleProfiling=null, 
    testFunctionalTest=null, 
    signingConfig=null, 
    resConfig=null, 
    mBuildConfigFields={}, 
    mResValues={}, 
    mProguardFiles=[], 
    mConsumerProguardFiles=[], 
    mManifestPlaceholders={}, 
    mWearAppUnbundled=null
}
```
