invoke-super                                          调用 parent 方法;  
invokeinterface                                     调用接口方法, 运行时才确定一个实现此接口的对象;  
invoke-static                                          调用静态方法的, 编译时确认;  
invoke-virtual                                        虚方法调用, 和实例引用的实际对象有关, 方法运行时确认实际调用, 动态确认的;  
                                                               一般是带有修饰符 protected 或 public 的方法;  
invoke-direct                                          没有被覆盖方法的调用, 一般是 private 或 <init> 方法;         
.local v0, "name":Ljava/lang/String;     从内存中读取 v0 的数值, v0 的变量名叫"name", v0 的数据类型是 String;  
.local v0, "a":I                                        从内存中读取 v0 的数值, v0 的变量名叫"a", v0 的数据类型是 int;  
move-result vAA                                    将上一个 invoke 类型的指令操作的单字非对象结果负责 vAA;  
move-result-object vAA                        将上一个 invoke 类型指令操作的对象赋值给 vAA;  
move-exception vAA                             保存一个运行时发生的异常vAA寄存器，必须是异常发生时的异常处理的第一条指令;  
const-string v2, "A"                               将字符串常量 "A" 赋值给 v2;  
const/16 v3, 0x929                                把 0x929  赋值给 v3, 0x929 是 16 位数值;  
const/4 v1, 0x7                                      把 0x7  赋值给 v1, 0x7 是 4 位数值;  
return-void                                            返回一个void; 
return vAA                                             返回一个32位非对象类型的值, 返回寄存器为8位;
return-wide vAA                                    返回一个64位非对象类型的值, 返回寄存器为8位;  
return-object vAA                                  返回一个对象类型;    
monitor-enter vAA                                为指定的对象获取锁;  
monitor-exit vAA                                   释放指定的对象的锁;  
direct method 是指private函数;  
virtual method 是指 protected和 public 函数;  

### 参考  
https://www.jianshu.com/p/730c6e3e21f6  


