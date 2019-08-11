robust 框架有两个插件, robust 负责生成钩子函数, auto-patch-plugin 负责生成 patch 包;  
在编译过程中, robust 会给所有的类, 添加属性 changeQuickRedirect;  
每一个方法在运行之前, 都会判断 changeQuickRedirect 是否为空;  
    如果没有任何变化, 之继续执行自己的内部实现;  
    如果需要执行补丁包, 会为当前类生成 patch 类, 例如 MainActivityPatch 类, 执行 patch 类对应的新方法逻辑;  
如果 patch 类, 需要引用本类的方法或者字段, 都是通过反射来实现的, 这部分实现逻辑是 auto-patch-plugin 插件自动完成的;  
另外, 被访问的方法的权限, 也被修改成 public;  
下载制定的 patch 包, 并且利用 DexClassLoader 将 patch.dex 加载到内存, 并且初始化 patch.dex 中的 class, 再初始化需要修改的类的 changeQuickRedirect 字段;   

### 加载 patch  
PatchExecutor.run  
PatchExecutor.applyPatchList  
PatchExecutor.patch  
遍历每一个 dex 下的所有类文件, 如果需要热更新, 则会存在 ChangeQuickRedirect 的实现类, 并通过反射的形式, 给当前类的 changeQuickRedirect 属性赋值;  

### 运行 patch  
❀ 本类  
```
public class RobustActivity extends AppCompatActivity implements OnClickListener {
    public static ChangeQuickRedirect k;
    TextView l = null;
    Button m = null;

    @Add
    private String k() {
        return "Robust Modify!";
    }

    public void onClick(View view) {
        if (!PatchProxy.proxy(new Object[]{view}, this, k, false, 15, new Class[]{View.class}, Void.TYPE).isSupported && view.getId() == R.id.btnModify) {
            Toast.makeText(this, "button on click", 1).show();
        }
    }


    //     @Modify
    //     @Override
    //     protected void onCreate (Bundle savedInstanceState)
    
    @Modify
    public void onCreate(Bundle bundle) {
        if (!PatchProxy.proxy(new Object[]{bundle}, this, k, false, 14, new Class[]{Bundle.class}, Void.TYPE).isSupported) {
            super.onCreate(bundle);
            setContentView((int) R.layout.activity_robust);
            this.l = (TextView) findViewById(R.id.tvTitle);
            this.m = (Button) findViewById(R.id.btnModify);
            this.m.setOnClickListener(this);
            this.l.setText(k());
        }
    }
}
```
❀ patch 类  
```

public class RobustActivityPatch {
    RobustActivity originClass;

    public RobustActivityPatch(Object obj) {
        this.originClass = (RobustActivity) obj;
    }

    public static void staticRobustonCreate(RobustActivityPatch robustActivityPatch, RobustActivity robustActivity, Bundle bundle) {
        RobustActivityPatchRobustAssist.staticRobustonCreate(robustActivityPatch, robustActivity, bundle);
    }
    
    
    /* access modifiers changed from: protected */
    public void onCreate(Bundle savedInstanceState) {
        RobustActivityPatch robustActivityPatch;
        RobustActivityPatch robustActivityPatch2;
        RobustActivityPatch robustActivityPatch3;
        RobustActivityPatch robustActivityPatch4;
        staticRobustonCreate(this, this.originClass, savedInstanceState);
        EnhancedRobustUtils.invokeReflectMethod("setContentView", this == this ? this.originClass : this, getRealParameter(new Object[]{new Integer(2131296285)}), new Class[]{Integer.TYPE}, c.class);
        if (this == this) {
            robustActivityPatch = this.originClass;
        } else {
            robustActivityPatch = this;
        }
        View view = (View) EnhancedRobustUtils.invokeReflectMethod("findViewById", robustActivityPatch, getRealParameter(new Object[]{new Integer(2131165332)}), new Class[]{Integer.TYPE}, c.class);
        EnhancedRobustUtils.setFieldValue("l", this instanceof RobustActivityPatch ? this.originClass : this, view == this ? ((RobustActivityPatch) view).originClass : (TextView) view, RobustActivity.class);
        if (this == this) {
            robustActivityPatch2 = this.originClass;
        } else {
            robustActivityPatch2 = this;
        }
        View view2 = (View) EnhancedRobustUtils.invokeReflectMethod("findViewById", robustActivityPatch2, getRealParameter(new Object[]{new Integer(2131165220)}), new Class[]{Integer.TYPE}, c.class);
        EnhancedRobustUtils.setFieldValue("m", this instanceof RobustActivityPatch ? this.originClass : this, view2 == this ? ((RobustActivityPatch) view2).originClass : (Button) view2, RobustActivity.class);
        if (this instanceof RobustActivityPatch) {
            robustActivityPatch3 = this.originClass;
        } else {
            robustActivityPatch3 = this;
        }
        RobustActivity robustActivity = (Button) EnhancedRobustUtils.getFieldValue("m", robustActivityPatch3, RobustActivity.class);
        if (robustActivity == this) {
            robustActivity = ((RobustActivityPatch) robustActivity).originClass;
        }
        EnhancedRobustUtils.invokeReflectMethod("setOnClickListener", robustActivity, getRealParameter(new Object[]{this}), new Class[]{OnClickListener.class}, View.class);
        if (this instanceof RobustActivityPatch) {
            robustActivityPatch4 = this.originClass;
        } else {
            robustActivityPatch4 = this;
        }
        RobustActivity robustActivity2 = (TextView) EnhancedRobustUtils.getFieldValue("l", robustActivityPatch4, RobustActivity.class);
        RobustActivity robustActivity2 = (TextView) EnhancedRobustUtils.getFieldValue("l", robustActivityPatch4, RobustActivity.class);
        String str = (String) EnhancedRobustUtils.invokeReflectMethod("k", this == this ? this.originClass : this, new Object[0], null, RobustActivity.class);
        if (robustActivity2 == this) {
            robustActivity2 = ((RobustActivityPatch) robustActivity2).originClass;
        }
        EnhancedRobustUtils.invokeReflectMethod("setText", robustActivity2, getRealParameter(new Object[]{str}), new Class[]{CharSequence.class}, TextView.class);
        if (this instanceof RobustActivityPatch) {
            robustActivityPatch5 = this.originClass;
        } else {
            robustActivityPatch5 = this;
        }
        RobustActivity robustActivity3 = (TextView) EnhancedRobustUtils.getFieldValue("l", robustActivityPatch5, RobustActivity.class);
        String str2 = (String) EnhancedRobustUtils.invokeReflectMethod("RobustPublicnewFoo", new RobustActivityInLinePatch(getRealParameter(new Object[]{this})[0]), getRealParameter(new Object[0]), null, null);
        if (robustActivity3 == this) {
            robustActivity3 = ((RobustActivityPatch) robustActivity3).originClass;
        }
        EnhancedRobustUtils.invokeReflectMethod("setText", robustActivity3, getRealParameter(new Object[]{str2}), new Class[]{CharSequence.class}, TextView.class);
        EnhancedRobustUtils.invokeReflectMethod("RobustPublicpri", new RobustActivityInLinePatch(getRealParameter(new Object[]{this})[0]), getRealParameter(new Object[0]), null, null);        
    }

    public Object[] getRealParameter(Object[] objArr) {
        if (objArr == null || objArr.length < 1) {
            return objArr;
        }
        Object[] objArr2 = new Object[objArr.length];
        int i = 0;
        while (true) {
            int i2 = i;
            if (i2 >= objArr.length) {
                return objArr2;
            }
            if (objArr[i2] instanceof Object[]) {
                objArr2[i2] = getRealParameter(objArr[i2]);
            } else if (objArr[i2] == this) {
                objArr2[i2] = this.originClass;
            } else {
                objArr2[i2] = objArr[i2];
            }
            i = i2 + 1;
        }
    }

}
```
❀ InLinePatch 类  
对于新增方法, robust 又是怎么实现的呢, 事实上是通过新增一个  MainActivityInLinePatch 类来实现;  
```
public class RobustActivityInLinePatch {
    RobustActivity originClass;

    public RobustActivityInLinePatch(Object obj) {
        this.originClass = (RobustActivity) obj;
    }

    public String RobustPublicnewFoo() {
        return newFoo();
    }

    public void RobustPublicpri() {
        pri();
    }

    public Object[] getRealParameter(Object[] objArr) {
        if (objArr == null || objArr.length < 1) {
            return objArr;
        }
        Object[] objArr2 = new Object[objArr.length];
        int i = 0;
        while (true) {
            int i2 = i;
            if (i2 >= objArr.length) {
                return objArr2;
            }
            if (objArr[i2] instanceof Object[]) {
                objArr2[i2] = getRealParameter(objArr[i2]);
            } else if (objArr[i2] == this) {
                objArr2[i2] = this.originClass;
            } else {
                objArr2[i2] = objArr[i2];
            }
            i = i2 + 1;
        }
    }

    private String newFoo() {
        ((Integer) EnhancedRobustUtils.invokeReflectStaticMethod("w", Log.class, getRealParameter(new Object[]{"tag", "getModify"}), new Class[]{String.class, String.class})).intValue();
        ((Integer) EnhancedRobustUtils.invokeReflectStaticMethod("w", Log.class, getRealParameter(new Object[]{"tag", "getModify"}), new Class[]{String.class, String.class})).intValue();
        ((Integer) EnhancedRobustUtils.invokeReflectStaticMethod("w", Log.class, getRealParameter(new Object[]{"tag", "getModify"}), new Class[]{String.class, String.class})).intValue();
        for (int i = 0; i < 20; i++) {
            ((Integer) EnhancedRobustUtils.invokeReflectStaticMethod("w", Log.class, getRealParameter(new Object[]{"tag", "getModify"}), new Class[]{String.class, String.class})).intValue();
        }
        return "Robust Modify!";
    }

    private void pri() {
        RobustActivityInLinePatch robustActivityInLinePatch;
        RobustActivityInLinePatch robustActivityInLinePatch2;
        if (this instanceof RobustActivityInLinePatch) {
            robustActivityInLinePatch = this.originClass;
        } else {
            robustActivityInLinePatch = this;
        }
        RobustActivity robustActivity = (TextView) EnhancedRobustUtils.getFieldValue("l", robustActivityInLinePatch, RobustActivity.class);
        String str = "hello";
        if (robustActivity == this) {
            robustActivity = ((RobustActivityInLinePatch) robustActivity).originClass;
        }
        EnhancedRobustUtils.invokeReflectMethod("setText", robustActivity, getRealParameter(new Object[]{str}), new Class[]{CharSequence.class}, TextView.class);
        for (int i = 0; i < 20; i++) {
            if (this instanceof RobustActivityInLinePatch) {
                robustActivityInLinePatch2 = this.originClass;
            } else {
                robustActivityInLinePatch2 = this;
            }
            RobustActivity robustActivity2 = (TextView) EnhancedRobustUtils.getFieldValue("l", robustActivityInLinePatch2, RobustActivity.class);
            String str2 = "hello";
            if (robustActivity2 == this) {
                robustActivity2 = ((RobustActivityInLinePatch) robustActivity2).originClass;
            }
            EnhancedRobustUtils.invokeReflectMethod("setText", robustActivity2, getRealParameter(new Object[]{str2}), new Class[]{CharSequence.class}, TextView.class);
        }
        if (this == this) {
            this = this.originClass;
        }
        EnhancedRobustUtils.invokeReflectMethod("finish", this, new Object[0], null, Activity.class);
    }
}
```
class 访问权限问题;  
super 方法调用的问题, 字节码指令 invoke vertical 调用的是普通方法, invoke super 调用的是 super 方法 ;  
```
gradlew clean assembleRelease --stacktrace --no-daemon  
gradle clean assembleRelease --stacktrace --no-daemon  
./gradlew clean assembleRelease --stacktrace --no-daemon  
```

### 优缺点  
由于使用多 ClassLoader 方案, 补丁中无新增 Activity, 所以不算激进类型的动态加载, 无需 hook system;  
兼容性和稳定性更好, 不存在与校验问题;  
由于采用 InstantRun 的热更新机制, 所以可以即时生效, 不需要重启;  
对性能影响较小, 不需要合成 patch;  
支持方法级别的修复, 支持静态方法;  
支持新增方法和类;  
支持ProGuard的混淆, 内联, 编译器优化后引起的问题(桥方法, lambda, 内部类等)等操作;  
暂时不支持新增字段, 但可以通过新增类解决;  
暂时不支持修复构造方法, 已经在内测;  
暂时不支持资源和 so 修复, 不过这个问题不大, 因为独立于 dex 补丁, 已经有很成熟的方案了, 就看怎么打到补丁包中以及 diff 方案;  
对于返回值是 this 的方法支持不太好;  
没有安全校验, 需要开发者在加载补丁之前自己做验证;  
可能会出现深度方法内联导致的不可预知的错误(几率很小可以忽略);  
### 参考  
key word  robust 源码分析  
http://w4lle.com/2017/03/31/robust-0/  
http://w4lle.com/2018/05/28/robust-1/  
作者的付费讲解资料  
https://www.zhihu.com/lives/772803407386800128   


用法  
https://www.jianshu.com/p/d51435895b79  
https://github.com/GaoXiaoduo/gxd-robust  
https://blog.csdn.net/ljw124213/article/details/73844811  
https://blog.csdn.net/junhuahouse/article/details/72465893  

废物  
https://www.cnblogs.com/yrstudy/p/8977315.html  
http://www.520monkey.com/archives/934  
https://blog.csdn.net/jiangwei0910410003/article/details/53705040  

