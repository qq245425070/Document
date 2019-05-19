### AppBar  
一个Material Design应用程序栏，由工具栏和其他可能的widget（如TabBar和FlexibleSpaceBar）组成。  

### SliverAppBar  
```
const SliverAppBar({
    Key key,
    this.leading,         //左侧标题
    this.automaticallyImplyLeading = true,
    this.title,               //标题
    this.actions,          //菜单
    this.flexibleSpace,        //可以展开区域，通常是一个FlexibleSpaceBar
    this.bottom,         //底部内容区域
    this.elevation,            //阴影
    this.forceElevated = false, 
    this.backgroundColor,       //背景色
    this.brightness,   /主题明亮
    this.iconTheme,  //图标主题
    this.textTheme,    //文字主题
    this.primary = true,  //是否预留高度
    this.centerTitle,     //标题是否居中
    this.titleSpacing = NavigationToolbar.kMiddleSpacing,
    this.expandedHeight,     //展开高度
    this.floating = false,       //是否随着滑动隐藏标题
    this.pinned = false,  //是否固定在顶部
    this.snap = false,   //与floating结合使用
  })
  

SliverAppBar(
  leading: IconButton(
      icon: Icon(
        Icons.arrow_back,
        color: Color(0xFFFFFFFF),
      ),
      onPressed: () {
        Router.pop(appContext);
      }),
  elevation: 20,
  pinned: true,
  floating: true,
  expandedHeight: 200.0,
  flexibleSpace: FlexibleSpaceBar(
    title: Text('Demo'),
    collapseMode: CollapseMode.pin,
    //  标题居中
    centerTitle: false,
  ),
),

```
### 参考  
https://docs.flutter.io/flutter/material/AppBar-class.html  
