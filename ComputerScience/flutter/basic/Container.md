### Container  
Container 可让您创建矩形视觉元素。container 可以装饰为一个BoxDecoration, 如 background、一个边框、或者一个阴影。  
 Container 也可以具有边距（margins）、填充(padding)和应用于其大小的约束(constraints)。另外， Container可以使用矩阵在三维空间中对其进行变换。  
 
### 相对布局  
```
var container2 = Container(
  height: 56,
  color: Colors.blueAccent,
  child: Stack(
    children: <Widget>[
      Align(
        alignment: FractionalOffset(0.5, 0.5),
        child: Text("center"),
      ),
      Align(
        alignment: FractionalOffset(0.0, 0.5),
        child: Container(
          child: Text("left"),
          margin: EdgeInsets.only(left: 16.0),
        ),
      ),
      Align(
        alignment: FractionalOffset(1.0, 0.5),
        child: Container(
          child: Text("right"),
          margin: EdgeInsets.only(right: 16.0),
        ),
      ),
    ],
  ),
);
```
### 参考  
https://docs.flutter.io/flutter/widgets/Container-class.html  
