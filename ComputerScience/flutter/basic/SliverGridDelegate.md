SliverGridDelegateWithMaxCrossAxisExtent  
```
  gridDelegate: SliverGridDelegateWithMaxCrossAxisExtent(
    //  最大副轴长度
    maxCrossAxisExtent: 200.0,
    //  主轴方向的间距, 垂直间距
    mainAxisSpacing: 10.0,
    //  横轴方向子元素的间距, 水平间距
    crossAxisSpacing: 10.0,
    //  副轴除以主轴长度的比例
    childAspectRatio: 4.0,
  ),
```
SliverGridDelegateWithFixedCrossAxisCount  
```
  gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
    //  最大副轴长度
    crossAxisCount: 3,
    //  主轴方向的间距, 垂直间距
    mainAxisSpacing: 10.0,
    //  横轴方向子元素的间距, 水平间距
    crossAxisSpacing: 10.0,
    //  副轴除以主轴长度的比例
    childAspectRatio: 4.0,
  ),
```
SliverFixedExtentList  
```
SliverFixedExtentList(
  itemExtent: 50.0,
  delegate: SliverChildBuilderDelegate((BuildContext context, int index) {
    return Container(
      alignment: Alignment.center,
      color: Colors.lightBlue[100 * (index % 9)],
      child: Text('list item $index'),
    );
  },
      // 50个列表项, 如果不设置, 代表可以无限滑动
      childCount: 10,
      //  设置为false来禁用AutomaticKeepAlives
      addAutomaticKeepAlives: true),
),
```

ListView  
```
var listView = new ListView(
  shrinkWrap: true,
  padding: const EdgeInsets.all(20.0),
  children: <Widget>[new Text("hello"), new Text("hello1"), new Text("hello3")],
);
```