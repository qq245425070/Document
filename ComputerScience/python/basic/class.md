### 类

### 属性访问权限  
````  
class UserEntity:
    # 共有属性
    name = 'alex'
    # _ _ 私有属性
    __age = 20
    # _ protected 属性
    _pro = '你猜'

    def __init__(self, name, age) -> None:
        super().__init__()
        self.name = name
        self.__age = age
        
    # noinspection PyMethodMayBeStatic
    def printMessage(self):
        print('hello')

````  
### 临时扩展字段  
临时扩展字段，是针对于当前类对象的，而不是针对于当前类的；  
```
user.extensionField0 = '扩展字段'
print(user.extensionField0)

user2 = UserEntity('jook', 9)
# AttributeError: 'UserEntity' object has no attribute 'extensionField0'
try:
    # noinspection PyUnresolvedReferences
    print(user2.extensionField0)
except AttributeError:
    print("""'UserEntity' object has no attribute 'extensionField0'""")
```
### 类继承与函数重载  
```
class ChildEntity(UserEntity):
    def __init__(self, name, age) -> None:
        super().__init__(name, age)

    def printMessage(self):
        print('你好')
```

### 属性访问
```
__foo__: 定义的是特殊方法，一般是系统定义名字 ，类似 __init__() 之类的。

_foo: 以单下划线开头的表示的是 protected 类型的变量，即保护类型只能允许其本身与子类进行访问，不能用于 from module import *

__foo: 双下划线的表示的是私有类型(private)的变量, 只能是允许这个类本身进行访问了。
```

### 函数  

### 定义一个函数  

```
def printName(userName):
    print(userName)
    return


printName('alex')

```

### 传参数
参数传递与修改，这一点和java是一致的，基本数据类型，是传值；结构体，是传指针；  
支持  具名参数、缺省参数、不定长参数、匿名函数、偏函数   

### 可变参数
```
def calc_sum(*args)
``` 







