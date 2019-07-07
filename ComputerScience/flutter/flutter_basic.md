[配置SDK](basic/Start.md)    
打开studio 下载Flutter plug-in;    
[StatelessWidget](basic/StatelessWidget.md)  
StatefulWidget;  
监测页面生命周期;  
渲染监测;  页面监测;  
[链接](basic/StatefulWidget.md)  

SliverAppBar;  AppBar;  
[链接](basic/AppBar.md)  

SliverGridDelegate;  
SliverGridDelegateWithMaxCrossAxisExtent;  
SliverGridDelegateWithFixedCrossAxisCount;  
SliverFixedExtentList;  ListView;  
[链接](basic/SliverGridDelegate.md)  

AnimationController;  Animation;  Tween;   
[链接](basic/Animation.md)  

相对布局;  Container;  
[链接](basic/Container.md)  

[GestureDetector](basic/GestureDetector.md)  
[Scaffold](basic/Scaffold.md)  
[Text](basic/Text.md)  
[Placeholder](basic/Placeholder.md)  
[FlutterLogo](basic/FlutterLogo.md)  
[IconButton](basic/IconButton.md)  
[RaisedButton](basic/RaisedButton.md)  
[Row Column](basic/Row_Column.md)  

[BoxDecoration](basic/BoxDecoration.md)  


### Widget 分类  
Container            	只有一个子 Widget, 默认充满, 包含了padding, margin, color, 宽高, decoration 等配置;  
Padding	                只有一个子 Widget, 只用于设置Padding, 常用于嵌套child, 给child设置padding;  
Center                     只有一个子 Widget, 只用于居中显示, 常用于嵌套child, 给child设置居中;  
Stack                        可以有多个子 Widget, 子Widget堆叠在一起;  
Column                   可以有多个子 Widget, 代表一列, 垂直布局;  
Row                         可以有多个子 Widget, 代表一行, 水平布局;  
Expanded               只有一个子 Widget, 在  Column 和  Row 中充满;  
ListView                  可以有多个子 Widget;  
MaterialApp       	一般作为APP顶层的主页入口, 可配置主题, 多语言, 路由等;  
Scaffold	                一般用户页面的承载Widget, 包含appbar, snackbar, drawer 等;  
Appbar	                一般用于Scaffold的appbar , 内有标题, 二级页面返回按键等, 当然不止这些, tabbar等也会需要它;  
Text	                        显示文本, 几乎都会用到, 主要是通过style设置TextStyle来设置字体样式等;  
RichText	               富文本, 通过设置TextSpan, 可以拼接出富文本场景;  
TextField	               文本输入框: new TextField(controller: //文本控制器, obscureText: "hint文本");  
Image	                   图片加载: new FadeInImage.assetNetwork( placeholder: "预览图", fit: BoxFit.fitWidth, image: "url");  
FlatButton	           按键点击: new FlatButton(onPressed: () {},child: new Container());  
Offstage                  offstage: true 隐藏;  当不可见的时, 如果有动画等, 需要手动停掉, Offstage 并不会停掉动画等操作;  
Alignment               参数x: -1 最左边, 0 中间, 1 最右边;  参数y: -1 最上边, 0 中间, 1 最下边; 
FractionalOffset     0,0 在左上角; 1,1 在右下角;  

### 参考  
https://github.com/crazycodeboy/awesome-flutter-cn  
https://livebook.manning.com/#!/book/flutter-in-action/welcome/v-7/  
https://github.com/flutter/flutter/releases  
https://book.flutterchina.club/  
https://www.itcodemonkey.com/article/11041.html   


Flutter中文  
https://flutterchina.club/widgets-intro/  
https://book.flutterchina.club/  

基础Widget  
http://examplecode.cn/categories/Flutter/  
https://flutterchina.club/widgets/basics/  
Material Components Widgets  
https://flutterchina.club/widgets/material/  
Cupertino(iOS风格的widget)  
https://flutterchina.club/widgets/cupertino/  
布局 Widget  
https://flutterchina.club/widgets/layout/  
文本 Widget  
https://flutterchina.club/widgets/text/  
Assets;  图片;  Icons  
https://flutterchina.club/widgets/assets/  
表单 Widgets  
https://flutterchina.club/widgets/input/  
动画和Motion  
https://flutterchina.club/widgets/animation/  
交互模型Widget  
https://flutterchina.club/widgets/interaction/  

Layout Widgets  
https://flutter.io/widgets/layout/  
开源库  
https://pub.dartlang.org/flutter/  

样式 Widget  
https://flutterchina.club/widgets/styling/  
绘制和视觉效果  
https://flutterchina.club/widgets/painting/  
异步  
https://flutterchina.club/widgets/async/  
可滚动的Widget  
https://flutterchina.club/widgets/scrolling/   
辅助功能  
https://flutterchina.club/widgets/accessibility/  
https://yq.aliyun.com/articles/651005  
https://www.jianshu.com/p/39575a90e820  

Flutter完整开发实战详解(三、 打包与填坑篇)  
https://juejin.im/post/5b6fd4dc6fb9a0099e711162  
Flutter完整开发实战详解(四、 Redux、主题、国际化)  
https://juejin.im/post/5b79767ff265da435450a873  
Flutter完整开发实战详解(五、 深入探索)  
https://juejin.im/post/5bc450dff265da0a951f032b  
Flutter完整开发实战详解(六、 深入Widget原理)  
https://juejin.im/post/5c7e853151882549664b0543  
Flutter完整开发实战详解(七、 深入布局原理)  
https://juejin.im/post/5c8c6ef7e51d450ba7233f51  
Flutter完整开发实战详解(八、 实用技巧与填坑)  
https://juejin.im/post/5c9e328251882567b91e1cfb  
Flutter完整开发实战详解(九、 深入绘制原理)  
https://juejin.im/post/5ca0e0aff265da309728659a  
Flutter完整开发实战详解(十、 深入图片加载流程)  
https://juejin.im/post/5cb1896ce51d456e63760449  
Flutter完整开发实战详解(十一、全面深入理解Stream)  
https://juejin.im/post/5cc2acf86fb9a0321f042041  
Flutter完整开发实战详解(十二、全面深入理解状态管理设计)  
https://juejin.im/post/5cc816866fb9a03231209c7c  
Flutter完整开发实战详解(十三、全面深入触摸和滑动原理)  
https://juejin.im/post/5cd54839f265da03b2044c32  


深入理解Flutter Platform Channel  
https://www.jianshu.com/p/39575a90e820  



https://www.cnblogs.com/yangyxd/p/9168244.html  
https://juejin.im/post/5b699783e51d4533f5285ee7  
https://juejin.im/post/5b6cfcade51d451915575b7b  
https://juejin.im/post/5b6da4b2e51d45195c07a30f  
https://juejin.im/post/5b6fd4dc6fb9a0099e711162
https://juejin.im/post/5b631d326fb9a04fce524db2  
https://juejin.im/post/5b685a2a5188251ac22b71c0  
https://juejin.im/post/5b6915636fb9a04fd73a6ecf  
https://juejin.im/post/5b6bdb406fb9a04f89785cb5  
https://juejin.im/post/5b3c8a4be51d4519935860d5  
https://juejin.im/post/5b6111b3e51d45198905679a  
https://juejin.im/post/5b57e339e51d4519700f686b  
https://juejin.im/post/5b7383ab51882561131aad13  
https://juejin.im/post/5b5ed06b5188251aa30c790c  
https://juejin.im/post/5b5dc77fe51d451757327412  
https://juejin.im/post/5b456ebee51d4519277b7761  
https://juejin.im/post/5b73841951882561086e4906  
https://www.cnblogs.com/yangyxd/p/9232308.html  
https://zhuanlan.zhihu.com/p/36577285  
https://tech.meituan.com/waimai-flutter-practice.html  


