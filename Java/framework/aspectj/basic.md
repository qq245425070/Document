### 切入点  
以 execution  为例，最为简单；  
execution(修饰符 返回值  包.类.方法名(参数) throws异常)  
◑ 修饰符，一般省略，有  
public            公共方法  
\*                   任意  
◑ 返回值，不能省略，有  
void               返回没有值  
String            返回值字符串  
\*                  任意  
◑ 包，可以省略  
com.itheima.crm                  固定包  
com.itheima.crm.*.service     crm包下面子包任意（例如：com.itheima.crm.staff.service）  
com.itheima.crm..                crm包下面的所有子包（含自己）  
com.itheima.crm.*.service..   crm包下面任意子包，固定目录service，service目录任意包  
java.lang.String  匹配String类型  
java.*.String  
匹配java包下的任何“一级子包”下的String类型，如匹配java.lang.String，但不匹配java.lang.ss.String  
java..*  
匹配java包及任何子包下的任何类型,如匹配java.lang.String、java.lang.annotation.Annotation  
java.lang.*ing  
匹配任何java.lang包下的以ing结尾的类型  

java.lang.Number+  
匹配java.lang包下的任何Number的自类型，如匹配java.lang.Integer，也匹配java.math.BigInteger  

◑ 类，可以省略  
UserServiceImpl                  指定类  
\*Impl                                  以Impl结尾  
User*                                  以User开头  
\*                                        任意  
◑ 方法名，不能省略  
addUser                               固定方法  
add*                                   以add开头  
*Do                                    以Do结尾  
\*                                        任意  
◑ 参数  
()                                        无参  
(int)                                    一个整型  
(int,int)                              两个  
(..)                                      参数任意  
(..,java.lang.String)   表示匹配接受java.lang.String类型的参数结束，且其前边可以接受有任意个参数的方法；  
(java.lang.String,..)  表示匹配接受java.lang.String类型的参数开始，且其后边可以接受任意个参数的方法；  
(\*,java.lang.String) 表示匹配接受java.lang.String类型的参数结束，且其前边接受有一个任意类型参数的方法；  

◑ throws,可省略，一般不写。  

### JoinPoint  

getThis  
```
/**
 * <p> Returns the currently executing object.  This will always be
 * the same object as that matched by the <code>this</code> pointcut
 * designator.  Unless you specifically need this reflective access,
 * you should use the <code>this</code> pointcut designator to
 * get at this object for better static typing and performance.</p>
 *
 * <p> Returns null when there is no currently executing object available.
 * This includes all join points that occur in a static context.</p>
 */
```
得到当前被执行的类对象，

getTarget  


### 基础知识  

execution 用于匹配方法执行的连接点；  
within 用于匹配指定类型内的方法执行；  
this 用于匹配当前AOP代理对象类型的执行方法；注意是AOP代理对象的类型匹配，这样就可能包括引入接口也类型匹配；  
target 用于匹配当前目标对象类型的执行方法；注意是目标对象的类型匹配，这样就不包括引入接口也类型匹配；  
args 用于匹配当前执行的方法传入的参数为指定类型的执行方法；  
@within 用于匹配所以持有指定注解类型内的方法；  
@target 用于匹配当前目标对象类型的执行方法，其中目标对象持有指定的注解；  
@args 用于匹配当前执行的方法传入的参数持有指定注解的执行；  
@annotation 用于匹配当前执行方法持有指定注解的方法；    

类型匹配  
* 匹配任何数量字符  
.. 匹配任何数量字符的重复，如在类型模式中匹配任何数量子包；而在方法参数模式中匹配任何数量参数。  
+ 匹配指定类型的子类型；仅能作为后缀放在类型模式后边。   

java.lang.String 匹配String类型  

java.*.String  
匹配java包下的任何 一级子包 下的String类型，  
匹配java.lang.String，  
不匹配java.lang.ss.String  

java..*  
匹配java包及任何子包下的任何类型  
匹配java.lang.String、java.lang.annotation.Annotation  

java.lang.*ing  
匹配任何java.lang包下的以ing结尾的类型  

java.lang.Number+  
匹配java.lang包下的任何Number的自类型  
匹配java.lang.Integer  
匹配java.math.BigInteger  

注解匹配  
@Deprecated  
匹配被 @Deprecated 标记的方法  

匹配修饰符 public  private  protected  
匹配返回值类型  * 表示任意类型  
方法名匹配  * 表示模式匹配  

参数列表    
()  表示 没有入参  
(..) 接受任意个参数  
(.., java.lang.String) 表示匹配接受java.lang.String类型的参数结束，且其前边可以接受有任意个参数的方法  
(java.lang.String, ..) 表示匹配接受java.lang.String类型的参数开始，且其后边可以接受任意个参数的方法  
(*, java.lang.String) 表示匹配接受java.lang.String类型的参数结束，且其前边接受有一个任意类型参数的方法  

异常列表  
可选，以“throws 异常全限定名列表”声明，异常全限定名列表如有多个以“，”分割，如throws java.lang.IllegalArgumentException, java.lang.ArrayIndexOutOfBoundsException。  

组合切入点表达式  
AspectJ使用 且（&&）、或（||）、非（！）来组合切入点表达式。  
在Schema风格下，由于在XML中使用“&&”需要使用转义字符“&amp;&amp;”来代替之，所以很不方便，因此Spring ASP 提供了and、or、not来代替&&、||、！  

execution 实例  
@Pointcut("execution(* com.alex.appserver..* (..)) && @annotation(org.springframework.web.bind.annotation.RequestMapping)")  
匹配 com.alex.appserver 包下的  所有的类， 被 @RequestMapping 标记的所有的 方法  
@Pointcut("bean(*Controller) && @annotation(org.springframework.web.bind.annotation.RequestMapping)")  
匹配 以Contorllor 结束的类 的 加上 @RequestMapping 标记的 执行方法  

### 参考  


