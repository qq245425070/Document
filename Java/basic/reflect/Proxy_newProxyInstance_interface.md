### Proxy.newProxyInstance#动态代理接口  
假设为一个接口做动态代理, 因为接口是比较特殊的, 没有构造方法;  
```
public interface ApiService {
    public static final String BASE_URL = "https://";
    @POST("updateVersion/query")
    Observable<WrapperBean<Update>> checkUpdate(@QueryMap Map<String, String> params);
}
```

❀ Proxy.newProxyInstance  
```
@CallerSensitive
public static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h) throws IllegalArgumentException
{
    Objects.requireNonNull(h);
    // intfs 只有一个元素,  name = com.alex.network.ApiService  
    final Class<?>[] intfs = interfaces.clone();
    ...
    // 这里就是 在内存中 生成 .class 字节码 的地方, 要断点进去的
    Class<?> cl = getProxyClass0(loader, intfs);
    ...
    try {
        ...
        // 调用 java.lang.reflect.Constructor.newInstance  为 接口产生  实例化对象  
        return cons.newInstance(new Object[]{h});
    } catch (IllegalAccessException|InstantiationException e) {
    }
    ... 
}
```
❀ Proxy.getProxyClass0  
```
private static Class<?> getProxyClass0(ClassLoader loader,Class<?>... interfaces) {
    ...
    // 这里好像 没有什么,  接着断点进去  
    return proxyClassCache.get(loader, interfaces);
}
```
❀ WeakCache.get  
```
public V get(K key, P parameter) {
    ...
    //subKeyFactory.apply(key, parameter)  给 这个 待生成的 类, 算出一个 key, 之后在生成value   
    Object subKey = Objects.requireNonNull(subKeyFactory.apply(key, parameter));
    Supplier<V> supplier = valuesMap.get(subKey);
    Factory factory = null;  // factory 实现了 Supplier  接口  
    while (true) {
        if (supplier != null) {
            // supplier might be a Factory or a CacheValue<V> instance
            // 不停迭代,  直到  supplier 能取出来 value  
            // 事实上是,  内存中有一个valuesMap,  
            // valuesMap.putIfAbsent(subKey, factory); 往 valuesMap 中 存数据; 
            // valuesMap.get(subKey); 从 内存中取数据
            V value = supplier.get();  // factory 实现了 Supplier 接口,  这里 等同于 调用 factory 的 get 方法  
            if (value != null) {
                return value;
            }
        }
       
        if (factory == null) {
            //  给 这个 待生成的 类, 生成value
            factory = new Factory(key, parameter, subKey, valuesMap);
        }

        if (supplier == null) {
            supplier = valuesMap.putIfAbsent(subKey, factory);
            if (supplier == null) {
                // successfully installed Factory
                supplier = factory;
            }
            // else retry with winning supplier
        } else {
            if (valuesMap.replace(subKey, supplier, factory)) {
                // successfully replaced
                // cleared CacheEntry / unsuccessful Factory
                // with our Factory
                supplier = factory;
            } else {
                // retry with current supplier
                supplier = valuesMap.get(subKey);
            }
        }
    }
}
```
❀ Proxy.KeyFactory.apply  
```
@Override
public Object apply(ClassLoader classLoader, Class<?>[] interfaces) {
    switch (interfaces.length) {
        // 这个代理对象 就是 java.lang.reflect.Proxy.Key1 类对象 
        case 1: return new Key1(interfaces[0]); // the most frequent
        ...
    }
}
```

❀ WeakCache.Factory.get  
```
@Override
public synchronized V get() { // serialize access
    // re-check
    Supplier<V> supplier = valuesMap.get(subKey);
    if (supplier != this) {
        return null;
    }
    // else still us (supplier == this)

    // create new value
    V value = null;
    try {
        // 目标 interface 的实现类,  就在此时此刻 生成了  ,  短点进去  
        value = Objects.requireNonNull(valueFactory.apply(key, parameter));
    } finally {
        if (value == null) { // remove us on failure
            valuesMap.remove(subKey, this);
        }
    }
    CacheValue<V> cacheValue = new CacheValue<>(value);
    if (valuesMap.replace(subKey, this, cacheValue)) {
        // put also in reverseMap
        reverseMap.put(cacheValue, Boolean.TRUE);
    } else {
        throw new AssertionError("Should not reach here");
    }
    return value;
}
```

❀ Proxy.ProxyClassFactory.apply  
```
@Override
public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
    Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
    for (Class<?> intf : interfaces) {
        Class<?> interfaceClass = null;
        try {
            interfaceClass = Class.forName(intf.getName(), false, loader);
        } catch (ClassNotFoundException e) {
        }
       ...
    }
    // 此时时刻 应该说,  终于看到太阳了 
    byte[] proxyClassFile = ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags);
    try {
        return defineClass0(loader, proxyName, proxyClassFile, 0, proxyClassFile.length);
    } catch (ClassFormatError e) {
       
    }
}
```

❀ ProxyGenerator.generateProxyClass
```
public static byte[] generateProxyClass(final String var0, Class<?>[] var1, int var2) {
    ProxyGenerator var3 = new ProxyGenerator(var0, var1, var2);
    final byte[] var4 = var3.generateClassFile();
    if (saveGeneratedFiles) {
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
                try {
                    int var1 = var0.lastIndexOf(46);
                    Path var2;
                    if (var1 > 0) {
                        Path var3 = Paths.get(var0.substring(0, var1).replace('.', File.separatorChar));
                        Files.createDirectories(var3);
                        var2 = var3.resolve(var0.substring(var1 + 1, var0.length()) + ".class");
                    } else {
                        var2 = Paths.get(var0 + ".class");
                    }

                    Files.write(var2, var4, new OpenOption[0]);
                    return null;
                } catch (IOException var4x) {
                    throw new InternalError("I/O exception saving generated file: " + var4x);
                }
            }
        });
    }

    return var4;
}
```
❀ Proxy.defineClass0  
private static native Class<?> defineClass0(ClassLoader loader, String name,byte[] b, int off, int len);  

### InvocationHandler.invoke#触发机制   
❀ Proxy.newProxyInstance  
```
public static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h) throws IllegalArgumentException{
    Objects.requireNonNull(h);
    // 这里会生成 代理类  
    Class<?> cl = getProxyClass0(loader, intfs);
    ... 
    try {
        final Constructor<?> cons = cl.getConstructor(constructorParams);
        ...
        // InvocationHandler 对象，在这里被用到， 断点进去  
        return cons.newInstance(new Object[]{h});
    } catch (IllegalAccessException|InstantiationException e) {
        throw new InternalError(e.toString(), e);
    } 
}
```
❀ Constructor.newInstance  
```
public T newInstance(Object ... initargs) throws InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException {
    ... 
    ConstructorAccessor ca = constructorAccessor;   // read volatile
    if (ca == null) {
        // 这里 可以 看到 ca 就是 NativeConstructorAccessorImpl  
        ca = acquireConstructorAccessor();
    }
    // InvocationHandler 对象，在这里被用到， 断点进去  
    T inst = (T) ca.newInstance(initargs);
    return inst;
}
```
❀ sun.reflect.NativeConstructorAccessorImpl.newInstance  
```
public Object newInstance(Object[] var1) throws InstantiationException, IllegalArgumentException, InvocationTargetException {
    if (++this.numInvocations > ReflectionFactory.inflationThreshold() && !ReflectUtil.isVMAnonymousClass(this.c.getDeclaringClass())) {
        ConstructorAccessorImpl var2 = (ConstructorAccessorImpl)(new MethodAccessorGenerator()).generateConstructor(this.c.getDeclaringClass(), this.c.getParameterTypes(), this.c.getExceptionTypes(), this.c.getModifiers());
        this.parent.setDelegate(var2);
    }

    // InvocationHandler 对象，在这里被用到，
    return newInstance0(this.c, var1);
}
```
❀ private static native Object newInstance0(Constructor<?> var0, Object[] var1)  

### 为什么只能代理接口   
ProxyGenerator#generateConstructor  
```
private ProxyGenerator.MethodInfo generateConstructor() throws IOException {
    ProxyGenerator.MethodInfo var1 = new ProxyGenerator.MethodInfo("<init>", "(Ljava/lang/reflect/InvocationHandler;)V", 1);
    DataOutputStream var2 = new DataOutputStream(var1.code);
    this.code_aload(0, var2);
    this.code_aload(1, var2);
    var2.writeByte(183);
    var2.writeShort(this.cp.getMethodRef("java/lang/reflect/Proxy", "<init>", "(Ljava/lang/reflect/InvocationHandler;)V"));
    var2.writeByte(177);
    var1.maxStack = 10;
    var1.maxLocals = 2;
    var1.declaredExceptions = new short[0];
    return var1;
}
```
生成的代理类  
```
public final class $Proxy0 extends Proxy implements tagService{  
  private static Method m1;  
  private static Method m3;  
  private static Method m0;  
  private static Method m2;  
  public $Proxy0(InvocationHandler paramInvocationHandler) {  
     super(paramInvocationHandler);  
   }  
}
```

意思是, 动态代理生成的代理类, 默认继承于 Proxy 类, 因为 java 不支持多继承, 所以多态就用接口实现;  
为什么要继承于 Proxy, 因为需要使用其内部的 invocationHandler 属性;  
动态代理, 更多是对行为的代理, 而不是对属性和代码块的代理;  
