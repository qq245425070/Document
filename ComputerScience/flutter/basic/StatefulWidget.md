### StatefulWidget 有状态布局  
为了构建更复杂的体验 - 例如, 以更有趣的方式对用户输入做出反应 - 应用程序通常会携带一些状态;  
Flutter使用StatefulWidgets来满足这种需求;  
StatefulWidgets是特殊的widget, 它知道如何生成State对象, 然后用它来保持状态;  
你可能想知道为什么StatefulWidget和State是单独的对象, 在Flutter中, 这两种类型的对象具有不同的生命周期:    
Widget是临时对象, 用于构建当前状态下的应用程序, 而State对象在多次调用build()之间保持不变, 允许它们记住信息(状态);  

initState: 初始化, 理论上只有初始化一次, 第二篇中会说特殊情况下;  
didChangeDependencies: 在 initState 之后调用, 此时可以获取其他 State;  
dispose: 销毁, 只会调用一次;  

### 监测页面生命周期  
是FlutterActivity 的, 不是某个 Flutter page 的;  
```
class _HomeBodyState extends State<_HomeBody> with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this); //添加观察者
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    super.didChangeAppLifecycleState(state);
    LogTrack.w("state=" + state.toString());
  }

  @override
  void dispose() {
    super.dispose();
    WidgetsBinding.instance.removeObserver(this); //销毁
  }
 
}
```
### 渲染监测  
页面监测;  
```
WidgetsBinding.instance.addPostFrameCallback((_) {
  //  只在当前 Page 渲染完成, 回调一次;  
  onFrameRendered();
});
WidgetsBinding.instance.addPersistentFrameCallback((_){
  //  每次刷新 page 都会回调一次;  
  LogTrack.v("Frame has been rendered");
});
```
### 参考  
https://docs.flutter.io/flutter/widgets/StatefulWidget-class.html  
