robust 框架有两个插件, robust 负责生成钩子函数, auto-patch-plugin 负责生成 patch 包;  
在编译过程中, robust 会给所有的类, 添加属性 ChangeQuickRedirect changeQuickRedirect;  
每一个方法在运行之前, 都会判断 changeQuickRedirect 对象是否为空, 并且当前方法被加上 @Modify 注解;  
    如果没有任何变化, 之继续执行自己的内部实现;  
    如果需要执行补丁包, 会为当前类生成 patch 类, 例如 MainActivityPatch 类;  
并且不会执行原来的逻辑, 而是执行 patch 类对应的新方法;  
如果 patch 类, 需要引用本类的方法或者字段, 都是通过反射来实现的, 这部分实现逻辑是 auto-patch-plugin 插件自动完成的;  
另外, 被访问的方法的权限, 也被修改成 public;  
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
patch 类  
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
        String str = (String) EnhancedRobustUtils.invokeReflectMethod("k", this == this ? this.originClass : this, new Object[0], null, RobustActivity.class);
        if (robustActivity2 == this) {
            robustActivity2 = ((RobustActivityPatch) robustActivity2).originClass;
        }
        EnhancedRobustUtils.invokeReflectMethod("setText", robustActivity2, getRealParameter(new Object[]{str}), new Class[]{CharSequence.class}, TextView.class);
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
class 访问权限问题;  
super 方法调用的问题;  
```
gradlew clean assembleRelease --stacktrace --no-daemon  
gradle clean assembleRelease --stacktrace --no-daemon  
./gradlew clean assembleRelease --stacktrace --no-daemon  
```

### 参考  
key word  robust 源码分析  
http://w4lle.com/2017/03/31/robust-0/  

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

