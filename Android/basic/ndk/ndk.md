### ndk 环境  
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
