1.. 新建项目, 包含 C++环境;  
2.. 新建java类  
```
public class JniHelper {
    static {
	   /* moduleName  "HelloJni"  */
        System.loadLibrary("HelloJni");
    }
    /**
     * 从 jni-c 文件中获取 字符串，这时候有红色，不用管
     */
    public static native String getStringFromC();
}
```
2.. rebuild  
Build -> rebuild project  


### 参考  
ndk 版本  
https://developer.android.google.cn/ndk/downloads/index.html  
https://dl.google.com/android/repository/android-ndk-r13-linux-x86_64.zip  
https://dl.google.com/android/repository/android-ndk-r13b-linux-x86_64.zip  
https://dl.google.com/android/repository/android-ndk-r13b-windows-x86_64.zip  
