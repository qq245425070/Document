习惯和表达式  
_ 下划线代表 private, 修饰变量或者函数, 表示只在库内可见, 也有 @protected 注解;  
```
AA ?? "999"  ///表示如果 AA 为空，返回999
AA ??= "999" ///表示如果 AA 为空，给 AA 设置成 999
AA ~/999 ///AA 对于 999 整除
```

### 类  
```
class ModelA {
  String name;
  String tag;
  
  //默认构造方法，赋值给name和tag
  ModelA(this.name, this.tag);

  //返回一个空的ModelA
  ModelA.empty();
  
  //返回一个设置了name的ModelA
  ModelA.forName(this.name);
}
```
### 对 getter setter 的重写  
```
@override
Size get preferredSize {
    return Size.fromHeight(kTabHeight + indicatorWeight);
}
```
### 运算符重载  
```
<  >  >=  <=  
+  -  *  /  
%  >>  <<  &  ^  []  []=  ~  ==  ~/  |  

class Vector {
  final int x, y;
  Vector(this.x, this.y);
  Vector operator +(Vector v) => Vector(x + v.x, y + v.y);
  Vector operator -(Vector v) => Vector(x - v.x, y - v.y);
}

void main() {
  final v = Vector(2, 3);
  final w = Vector(2, 2);

  assert(v + w == Vector(4, 5));
  assert(v - w == Vector(0, 1));
}

```
### async/await  
```
 ///模拟等待两秒，返回OK
  request() async {
    await Future.delayed(Duration(seconds: 1));
    return "ok!";
  }

  ///得到"ok!"后，将"ok!"修改为"ok from request"
  doSomeThing() async {
    String data = await request();
    data = "ok from request";
    return data;
  }

  ///打印结果
  renderSome() {
    doSomeThing().then((value) {
      print(value);
      ///输出ok from request
    });
  }

```
### 命名参数  与 默认参数
命名参数必须 被 {} 包裹；  
默认参数必须是 常量 static const；  
```
TutorialHome({this.centerChild, this.backgroundColor: defaultBackgroundColor});
```

### 定义一个回调函数  
```
typedef void CartChangedCallback(Product product, bool inCart);
```

### 隐式接口  
```
class Person {
    final _name;
    Person(this._name);
    String greet(who) => "Hello, $who. I am $_name.";
}

class Imposter implements Person {
    final name = "";
    String greet(who) => "Hi $who. Do you know who I am?";
}
```