### Java相关知识  
基础知识;  基础概念;  代码规范;  
[链接](basic/library/basic_concept.md)  

接口 和 抽象类;    对Java中面向对象的认识;    Java类的各种成员初始化顺序;    
[链接](basic/library/interface_abstract_class.md)  
[java位运算](basic/library/BitOperation.md)  
[参数传递](basic/library/fun_params.md)  
[单元测试](basic/library/test_junit.md)  
[hashCode的问题](basic/library/java_hashCode.md)  
[String](basic/library/String.md)  
[Stream](basic/library/Stream.md)  
### Reflect;  反射机制;  动态代理;  
Proxy.newProxyInstance 动态代理  
```
ApiService apiService = Proxy.newProxyInstance(ClassLoader.getSystemClassLoader(), new Class[]{ApiService.class}, new InvocationHandler() {
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return null;
    }
});
```
概述就是, 利用反射, 解析这个 interface, 拿到关于这个 interface 的所有方法的声明, 在内存中生成一个实现这个接口的, 代理类;  
这个代理类所有的方法, 在调用之前, 都会先调用 invocationHandler.invoke 方法;  
所以每次调用代理对象的方法时, 都会触发相应的回调, 这时候能拿到, 代理对象, method 对象, 参数值的列表;  
[Proxy.newProxyInstance 动态代理接口](basic/reflect/Proxy_newProxyInstance_interface.md)  

### Class.forName 和 ClassLoader  
class.forName(className, initialize, classLoader);   
如果第二个参数 initialize 为 false, 则不会执行 类中的static 代码块, 为 true, 则会执行;  
事实上, 不只有 static 代码块会被执行, 而且 static 变量, 也会被赋值的;  
如果 static 变量, 需要依赖 另一个 static 函数, 给其赋值, 那么这个 static 函数, 也是会被执行的;  
classLoader 只干一件事情, 就是将.class 文件加载到 jvm 中, 不会执行 static 中的内容, 只有在 newInstance 才会去执行 static 块;  

### Error, Exception, Throwable  
在 Java 中, 根据错误性质将运行错误分为两类: 错误和异常;  
在 Java 程序的执行过程中, 如果出现了异常事件, 就会生成一个异常对象, 生成的异常对象将传递 Java 运行时系统, 异常的产生和提交过程称为抛弃异常;  
当 Java 运行时系统得到一个异常对象时, 它将会沿着方法的调用栈逐层回溯, 寻找处理这一异常的代码;  
找到能够处理这类异常的方法后, 运行时系统把当前异常对象, 交给这个方法进行处理, 这一过程称为捕获异常;  
Throwable 类是 Java 语言中所有错误或异常的超类, 它有两个子类 Error 和 Exception;  

Error 用于标识那些, 不应该被试图捕获的严重问题, 系统错误类, 如:  
内存溢出, 虚拟机错误, 栈溢出等, 这类错误一般与硬件有关, 与程序本身无关, 通常由系统进行处理, 程序本身无法捕获和处理;  

Exception 用于标识那些, 需要被程序捕获处理的问题, 为了保证程序的健壮性, 需要对那些 coding 阶段无法预料的问题, 进行捕获和处理;  

### System#arraycopy  
为什么, System#arraycopy 会比较快?  
native 实现, 时间复杂度是 O(n), 属于浅复制, 是内存地址的拷贝, 避免引用数据在内存中搬来搬去, 自然就会更快;  
在 int double 等基础数据类型上表现不明显, 因为有一些方法栈要调用, 甚至不一定比 for 循环速度快, 但是在引用数组的拷贝上, 表现很明显了;  

### 克隆  
基本数据类型, String 默认实现深克隆;  
浅克隆, 只是给原始引用数据多了一份指针, 多了一个对象的引用, 否则要实现 Cloneable 接口, 重写 clone 方法;  
引用数据类型, 深克隆后, 会出现2个对象, 修改新的对象数值, 对原始对象没有任何影响,   

❀ 浅克隆   
```
public class ShallowWordDocument implements Cloneable {
    private String text;
    private String author;
    private List<String> urlList = new ArrayList<>();
    public ShallowWordDocument cloneDocument() {
        try {
            ShallowWordDocument document = (ShallowWordDocument) super.clone();
            return document;
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
❀ 深克隆   
```
public class DeepWordDocument implements Cloneable {
    private String text;
    private String author;
    private ArrayList<String> urlList = new ArrayList<>();
    public DeepWordDocument cloneDocument() {
        try {
            DeepWordDocument document = (DeepWordDocument) super.clone();
            document.urlList = (ArrayList<String>) this.urlList.clone();
            return document;
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return null;
    }

}
```

### Java泛型  

基本概念: 泛型类,  泛型方法,    

好处  
避免了类型强转操作;  
避免一些方法重载;  

通配符  
? extends Number  可以接受Number以及其的子类, 称为上限通配符;   
? super  Number  可以接受Number以及其的父类, 称为下限通配符;   
? 只有一个?, 代表无限定通配符, 例如Class<?>方法;  

in与out  
// in 逆变  超类  
// out 协变 子类  
// 不变    

❀ 类型擦除  
将没有限定的类型参数用Object替换, 保证 class 文件中只含有正常的类,  接口与方法;  
在必要的时候进行类型转换, 保证类型安全;  
在泛型的继承上使用桥接方法（bridge methods）保持多态性;   
List<String>, List<T>  擦除后的类型为 List;  
List<String>[], List<T> [] 擦除后的类型为 List[];  
List<? extends E>, List<? super E> 擦除后的类型为 List<E>;  
List<T extends Serializable & Cloneable> 擦除后类型为 List<Serializable>;  
### Serializable  
当使用 ObjectInputStream, ObjectOutputStream 时, 需要使用标记接口 Serializable, 标记接口的目的, 是为了让其能转化成字节流保存起来,     
还能将本地的字节流再转成类对象, Serializable 主要应用于数据持久化-网络流;  

相关注意事项  
序列化时, 只对对象的状态进行保存, 而不管对象的方法;  
当一个父类实现序列化, 子类自动实现序列化, 不需要显式实现 Serializable 接口;  
当一个对象的实例变量引用其他对象, 序列化该对象时也把引用对象进行序列化;  
当某个字段被声明为 transient 后, 默认序列化机制就会忽略该字段;   
序列化的作用就是为了不同jvm之间共享实例对象的一种解决方案;  

### 同步和异步, 并发和并行  
并发是指在某个时间段内, 多任务交替处理的能力, 并行是指同时处理多任务的能力;  

❀ 同步和异步  
同步就是在发出一个功能调用的时候, 在没有得到响应之前, 该调用就不返回;  
异步就是在发出一个功能调用的时候, 不需要等待响应, 继续进行该做的事情, 一旦得到了响应, 会给出响应的处理;  

❀ 并发(concurrency)和并行(parallelism)  
在单 CPU 上, 不存在并行, 多线程编程, 只可能是轮流分配 CPU 时间片, 这种方式称为并发;  
在多 CPU 上, 两个线程运行在不同的 CPU 上, 互不干扰, 可以同时进行, 这种方式称为并行;  

并行是指两个或者多个事件在同一时刻发生, 而并发是指两个或多个事件在同一时间间隔发生;  
并行是在不同实体上的多个事件, 并发是在同一实体上的多个事件;    

并行, 同一时刻执行多个任务, 形式上可以是多线程乃至多进程, 被多个 CPU 同时运作, 任务之间无任何依赖关系, 不互斥,  不交叉;   
并发, 指程序逻辑上的一种能力, 支持多个任务同时存在一起被处理的能力; 一个任务通常被划分为一个个的时间分片, 比肩接踵般轮流的;  
          可交替切换的被一个或多个CPU所执行, 即不同任务的时间分片之间会有互斥,  等待, 一个任务可能因为CPU资源被抢走而被中断执行;   

### 命令行  
javap -c  classPath  
```
-help  --help  -?        输出此用法消息
-version                 版本信息
-v  -verbose             输出附加信息
-l                       输出行号和本地变量表
-public                  仅显示公共类和成员
-protected               显示受保护的/公共类和成员
-package                 显示程序包/受保护的/公共类
                       和成员 (默认)
-p  -private             显示所有类和成员
-c                       对代码进行反汇编
-s                       输出内部类型签名
-sysinfo                 显示正在处理的类的
                       系统信息 (路径, 大小, 日期, MD5 散列)
-constants               显示最终常量
-classpath <path>        指定查找用户类文件的位置
-cp <path>               指定查找用户类文件的位置
-bootclasspath <path>    覆盖引导类文件的位置

```
### 参考  
http://blog.csdn.net/u012152619/article/details/47253811  

System#arraycopy  
https://blog.csdn.net/jianghuxiaojin/article/details/53541930  
https://blog.csdn.net/wangyangzhizhou/article/details/79504818  

