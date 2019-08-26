### Class 类文件结构  
Class 文件是一组, 以一个字节(8 位)为基础单位的二进制流, 各个数据项目严格按照顺序紧凑地排列在 Class 文件之中, 中间没有添加任何分隔符,  
当遇到需要占用 8 位字节以上空间的数据项时, 则会按照高位在前的方式分割成若干个 8 位字节进行存储;  
根据 Java 虚拟机规范的规定, Class 文件格式采用一种类似于 C 语言结构体的伪结构来存储数据, 这种伪结构中只有两种数据类型: 无符号数和表;  

魔数与 Class 文件版本  
每个 Class 文件的头 4 个字节称为魔数(Magic Number), 它的唯一作用是确定这个文件是否为一个能被虚拟机接受的 Class 文件;  
很多文件存储标准中都使用魔数来进行身份识别, 譬如图片格式, 如 gif 或者 jpeg 等在文件头中都存有魔数;  
使用魔数而不是扩展名来进行识别, 主要是基于安全方面的考虑, 因为文件扩展名可以随意地改动;  
文件格式的制定者可以自由地选择魔数值, 只要这个魔数值还没有被广泛采用过同时又不会引起混淆即可;  
Class 文件的魔数的获得很有 "浪漫气息" , 值为: 0xCAFEBABE(咖啡宝贝?),  
这个魔数值在 Java 还称做 "Oak" 语言的时候(大约是 1991 年前后)就已经确定下来了;  

紧接着魔数的 4 个字节存储的是 Class 文件的版本号,  高版本的 JDK 能向下兼容以前版本的 Class 文件, 但不能运行以后版本的 Class 文件,  
即使文件格式并未发生任何变化, 虚拟机也必须拒绝执行超过其版本号的 Class 文件;  

访问标志  
在常量池结束之后, 紧接着的两个字节代表访问标志(access_flags), 这个标志用于识别一些类或者接口层次的访问信息,  
包括: 这个 Class 是类还是接口; 是否定义为 public 类型; 是否定义为 abstract 类型; 如果是类的话, 是否被声明为 final 等;  

类索引,  父类索引与接口索引集合  
Class 文件中由这三项数据来确定这个类的继承关系; 类索引用于确定这个类的全限定名, 父类索引用于确定这个类的父类的全限定名;  
由于 Java 语言不允许多重继承, 所以父类索引只有一个;  
除了 java.lang.Object 之外, 所有的 Java 类都有父类;  

接口索引集合就用来描述这个类实现了哪些接口, 这些被实现的接口将按 implements 语句后的接口顺序, 从左到右排列在接口索引集合中;  

字段表集合  
字段表(field_info)用于描述接口或者类中声明的变量, 字段(field)包括类级变量以及实例级变量, 但不包括在方法内部声明的局部变量;  

方法表集合  
方法表的结构如同字段表一样, 依次包括了:  
访问标志(access_flags),  名称索引(name_index),  描述符索引(descriptor_index),  属性表集合(attributes)  

属性信息  
如果字段有额外信息, 会在 attributes 里面出现  
方法的代码就是放在 attributes 的 Code 属性里面  
提供类的额外信息, 包括注解 RetentionPolicy.CLASS 或 RetentionPolicy.RUNTIME  
```
属性名                              使用位置                                              含义  
Code                                   方法表                                        Java 代码编译成的字节码指令  
ConstantValue                  字段表                                        final 关键字定义的常量值  
Deprecated                       类, 方法表, 字段表                    被声明为 deprecated 的方法和字段  
Exception                          方法表                                        方法抛出的异常  
EnclosingMethod             0x0010                                        方法是否为 final  
InnerClass                         类文件                                         内部类列表  
LineNumberTable            Code 属性                                   Java 源码的行数和字节码指令的对应关系  
LocalVariableTable          Code 属性                                   方法的局部变量描述  
SourceFile                          类文件                                        记录源文件名称  
```
### 参考  
http://www.it610.com/article/5901050.htm  
https://yq.aliyun.com/articles/663999  



