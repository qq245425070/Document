```
# 抑制警告
-ignorewarnings
# 代码混淆压缩比，在0~7之间，默认为5，一般不做修改
-optimizationpasses 5
# 提高优化步骤
-allowaccessmodification
# 混合时不使用大小写混合，混合后的类名为小写
-dontusemixedcaseclassnames
# 混淆时是否记录日志
# 这句话能够使我们的项目混淆后产生映射文件
# 包含有类名 -> 混淆后类名的映射关系
-verbose
# 优化  不优化输入的类文件
-dontoptimize
-dontobfuscate
# 不做预校验，preverify是proguard的四个步骤之一，Android不需要preverify，去掉这一步能够加快混淆速度。
-dontpreverify
# 不忽略非公共库的类
-dontskipnonpubliclibraryclasses
# 指定不去忽略非公共库的类成员
-dontskipnonpubliclibraryclassmembers

-printconfiguration

# 避免混淆泛型
-keepattributes Signature
-keepattributes Signature-keep class * extends java.lang.annotation.Annotation {*;}
# 抛出异常时保留代码行号
-keepattributes SourceFile,LineNumberTable
-keepattributes *Annotation*,InnerClasses
#  保持内部类不被混淆
-keepattributes InnerClasses
-keepattributes Signature
#  保持反射不被混淆
-keepattributes EnclosingMethod
-printmapping proguardMapping.txt
# 指定混淆是采用的算法，后面的参数是一个过滤器
# 这个过滤器是谷歌推荐的算法，一般不做更改
-optimizations  !code/simplification/arithmetic,!code/simplification/cast,!field/*,!class/merging/*,!method/*
# -optimizations !code/simplification/cast,!field/*,!class/merging/*
#---------------------------------    Activity     开始     ↓    ------------------------------------

#---------------------------------    Activity     结束    ↑    ------------------------------------




#---------------------------------    R     开始     ↓    ------------------------------------
-keep class **.R$* {*;}
#---------------------------------    R     结束    ↑    ------------------------------------


#---------------------------------    Application     开始     ↓    ------------------------------------
-keep public class * extends android.app.Application
-keep public class * extends android.support.multidex.MultiDexApplication
#---------------------------------    Application     结束    ↑    ------------------------------------


#---------------------------------    四大组件     开始     ↓    ------------------------------------
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Appliction
-keep public class * extends android.app.Fragment
-keep public class * extends android.support.v4.app.Fragment
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.preference.Preference
-keep public class com.android.vending.licensing.ILicensingService
-keep public class com.google.vending.licensing.ILicensingService
#---------------------------------    四大组件     结束    ↑    ------------------------------------

#---------------------------------    support     开始     ↓    ------------------------------------
# 保留support下的所有类及其内部类

# 保留继承的
-dontwarn android.support.**
-keep class android.support.** {*;}
-keep public class * extends android.support.v4.**
-keep public class * extends android.support.v7.**
-keep public class * extends android.support.annotation.**
#---------------------------------    support     结束    ↑    ------------------------------------


#---------------------------------    support v4     开始     ↓    ------------------------------------

-keep class android.support.v4.** { *; }
-keep public class * extends android.support.v4.**
-keep public class * extends android.app.Fragment
-keepclasseswithmembernames class android.support.v4.widget.** { *; }
#---------------------------------    support v4     结束    ↑    ------------------------------------

#---------------------------------    support v7     开始     ↓    ------------------------------------
-keep class android.support.v7.internal.** { *; }
-keep interface android.support.v7.internal.** { *; }
-keep class android.support.v7.widget.** { *; }
-keep public class * extends android.support.v7.app.ActionBarActivity { *; }
-keep class android.support.annotation.Keep
-keep class androidx.annotation.Keep

-keep @android.support.annotation.Keep class * {*;}
-keep @androidx.annotation.Keep class * {*;}
# 保持 support Keep 类成员不被混淆
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <methods>;
}
# 保持 androidx Keep 类成员不被混淆
-keepclasseswithmembers class * {
    @androidx.annotation.Keep <methods>;
}
# 保持 support Keep 类成员不被混淆
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <fields>;
}
# 保持 androidx Keep 类成员不被混淆
-keepclasseswithmembers class * {
    @androidx.annotation.Keep <fields>;
}
# 不混淆所有类及其类成员中的使用注解的初始化方法
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <init>(...);
}
# 不混淆所有类及其类成员中的使用注解的初始化方法
-keepclasseswithmembers class * {
    @androidx.annotation.Keep <init>(...);
}
-keep class * extends java.lang.annotation.Annotation { *; }

#  keep相关注解
-keep class android.support.annotation.Keep
-keep @android.support.annotation.Keep class * {*;}
#---------------------------------    support v7     结束    ↑    ------------------------------------


#---------------------------------    点击事件     开始     ↓    ------------------------------------
-keepclassmembers class *  {
    void *(android.view.View);
}
-keepclassmembers class * extends android.app.Activity {
   void *(android.view.View);
}
#---------------------------------    点击事件     结束    ↑    ------------------------------------

#---------------------------------    webview     开始     ↓    ------------------------------------
# 自定义的 WebView
-dontwarn android.webkit.WebView

-keepclassmembers class fqcn.of.javascript.interface.for.Webview {
   public *;
}
-keepclassmembers class * extends android.webkit.WebViewClient {
    public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);
    public boolean *(android.webkit.WebView, java.lang.String);
}
-keepclassmembers class * extends android.webkit.WebViewClient {
    public void *(android.webkit.WebView, jav.lang.String);
}

#---------------------------------    webview     结束    ↑    ------------------------------------


#---------------------------------    Log     开始     ↓    ------------------------------------
#  keep  是保持 log，没有 keep，就是 混淆 log
-keep class org.alex.util.LogTrack {*;}

#  去掉 日志 函数
-assumenosideeffects class android.util.Log {
    public static *** d(...);
    public static *** v(...);
    public static *** i(...);
    public static *** w(...);
}
#    public static *** e(...);
#---------------------------------    Log     结束    ↑    ------------------------------------

#---------------------------------    System.out.println     开始     ↓    ------------------------------------
-assumenosideeffects class java.io.PrintStream {
    public *** println(...);
    public *** print(...);
}
#---------------------------------    System.out.println     结束    ↑    ------------------------------------


#---------------------------------    自定义 view     开始     ↓    ------------------------------------
-keep public class * extends android.view.View
-keepclassmembers public class * extends android.view.View {
    public <init>(android.content.Context);
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
    void set*(***);
    *** get*();
}

-keepclasseswithmembers class * {
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
}

#---------------------------------    自定义 view     结束    ↑    ------------------------------------





#---------------------------------    Serializable     开始     ↓    ------------------------------------
#保持 Serializable 不被混淆
-keep class * implements java.io.Serializable { *; }
-keepnames class * implements java.io.Serializable
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    !private <fields>;
    !private <methods>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}
#---------------------------------    Serializable     结束    ↑    ------------------------------------

#---------------------------------    Parcelable     开始     ↓    ------------------------------------
-keep class * implements android.os.Parcelable { *; }
# 保持 Parcelable 不被混淆
-keep class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator *;
}
#---------------------------------    Parcelable     结束    ↑    ------------------------------------

#---------------------------------    native     开始     ↓    ------------------------------------
-keepclasseswithmembernames class * { native <methods>;}
-keepclasseswithmembers class * { native <methods>; }
#---------------------------------    native     结束    ↑    ------------------------------------

#---------------------------------    enum     开始     ↓    ------------------------------------
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
#---------------------------------    enum     结束    ↑    ------------------------------------


#---------------------------------    squareup  全家桶     开始     ↓    ------------------------------------

# retrofit2
-keep class retrofit2.** { *; }
-dontwarn retrofit2.**
-dontwarn javax.annotation.**

# 支持库包含对较新版本版本的引用。
# 不要警告那些情况下,这个应用程序链接到旧的
# 平台版本。我们知道他们是安全的。
-dontnote android.support.**
-dontnote androidx.**
-dontwarn android.support.**
-dontwarn androidx.**
# Platform calls Class.forName on types which do not exist on Android to determine platform.
-dontnote retrofit2.Platform
# Platform used when running on RoboVM on iOS. Will not be used at runtime.
-dontnote retrofit2.Platform$IOS$MainThreadExecutor
# Platform used when running on Java 8 VMs. Will not be used at runtime.
-dontwarn retrofit2.Platform$Java8
# Retain generic type information for use by reflection by converters and adapters.
-keepattributes Signature
# 保持异常不被混淆
-keepattributes Exceptions
#-------------- okhttp3 -------------
#okhttp
-dontwarn okhttp3.**
-keep class okhttp3.**{*;}
#okio
-dontwarn okio.**
-keep class okio.**{*;}
# 排除 android.jar 和 org.apache.http.legacy.jar 之间重复
-dontnote org.apache.http.**
-dontnote android.net.http.**
# ====== reactive ====================
-dontwarn io.reactivex.**
-keep class io.reactivex.**{ *;}
-dontwarn org.reactivestreams.**
-keep class org.reactivestreams.**{ *;}
-dontwarn com.trello.rxlifecycle2.**
-keep class com.trello.rxlifecycle2.**{ *;}
-keepclassmembers class rx.internal.util.unsafe.*ArrayQueue*Field* {
   long producerIndex;
   long consumerIndex;
}
#---------------------------------    squareup  全家桶     结束    ↑    ------------------------------------


#---------------------------------    butterknife     开始     ↓    ------------------------------------
-keep class butterknife.** { *; }
-dontwarn butterknife.internal.**
-keep class **$$ViewBinder { *; }

-keepclasseswithmembernames class * {
    @butterknife.* <fields>;
}

-keepclasseswithmembernames class * {
    @butterknife.* <methods>;
}
#---------------------------------    butterknife     结束    ↑    ------------------------------------


#---------------------------------    gson     开始     ↓    ------------------------------------
-keepattributes Signature
-keepattributes *Annotation*
-keep class sun.misc.Unsafe { *; }
-keep public class com.google.gson.** {*;}
-keep class com.google.gson.stream.** { *; }
# 使用Gson时需要配置Gson的解析对象及变量都不混淆。不然Gson会找不到变量。
# 将下面替换成自己的实体类
-keep class com.alex.*.entity.** { *; }
#---------------------------------    gson     结束    ↑    ------------------------------------


#---------------------------------    glide     开始     ↓    ------------------------------------
#========== glide v3.8  ===============
-dontwarn com.bumptech.glide.**
-keep class com.bumptech.glide.**{ *;}

-keep public class * implements com.bumptech.glide.module.com.bumptech.glide.annotation.GlideModule
-keep public enum com.bumptech.glide.load.resource.bitmap.ImageHeaderParser$** {
    **[] $VALUES;
    public *;
}

#========== glide v4.0 ===============
-keep public class * implements com.bumptech.glide.module.com.bumptech.glide.annotation.GlideModule
-keep public class * extends com.bumptech.glide.AppGlideModule
-keep public enum com.bumptech.glide.load.resource.bitmap.ImageHeaderParser$** {
  **[] $VALUES;
  public *;
}
# target API低于27
-dontwarn com.bumptech.glide.load.resource.bitmap.VideoDecoder
# for DexGuard only
#-keepresourcexmlelements manifest/application/meta-data@value=com.bumptech.glide.annotation.GlideModule
#---------------------------------    glide     结束    ↑    ------------------------------------



#---------------------------------    kotlin     开始     ↓    ------------------------------------
# https://stackoverflow.com/questions/33547643/how-to-use-kotlin-with-proguard
-dontwarn kotlin.**
-keep class kotlin.**{ *;}
-dontwarn kotlinx.**
-keep class kotlinx.**{ *;}
# Kotlin reflection directly.
-keep class kotlin.reflect.jvm.internal.impl.builtins.BuiltInsLoaderImpl
-keep class kotlin.reflect.jvm.internal.impl.load.java.FieldOverridabilityCondition
-keep class kotlin.reflect.jvm.internal.impl.load.java.ErasedOverridabilityCondition
-keep class kotlin.reflect.jvm.internal.impl.load.java.JavaIncompatibilityRulesOverridabilityCondition
# Companion objects
-if class **$Companion extends **
-keep class <2>
-if class **$Companion implements **
-keep class <2>
# https://medium.com/@AthorNZ/kotlin-metadata-jackson-and-proguard-f64f51e5ed32
-keep class kotlin.Metadata { *; }

-assumenosideeffects class kotlin.jvm.internal.Intrinsics {
    public static void checkParameterIsNotNull(java.lang.Object, java.lang.String);
    public static void checkNotNullParameter(java.lang.Object, java.lang.String);
    public static void checkExpressionValueIsNotNull(java.lang.Object, java.lang.String);
    public static void checkNotNullExpressionValue(java.lang.Object, java.lang.String);
    public static void checkReturnedValueIsNotNull(java.lang.Object, java.lang.String);
    public static void checkReturnedValueIsNotNull(java.lang.Object, java.lang.String, java.lang.String);
    public static void checkFieldIsNotNull(java.lang.Object, java.lang.String);
    public static void checkFieldIsNotNull(java.lang.Object, java.lang.String, java.lang.String);
    public static void throwUninitializedProperty(java.lang.String);
    public static void throwUninitializedPropertyAccessException(java.lang.String);
    public static void throwNpe(...);
}

#---------------------------------    kotlin     结束    ↑    ------------------------------------
```