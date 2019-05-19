Tween  
```
class _LoadingDialogState<LoadingDialog> extends State with SingleTickerProviderStateMixin {
  AnimationController controller;
  Animation<double> tween;

  @override
  void initState() {
    super.initState();
    controller = new AnimationController(vsync: this, duration: Duration(milliseconds: 500))
      ..addListener(() {
        setState(() {});
      });
    tween = new Tween(begin: 0.5, end: 1.0).animate(controller);
    controller.forward();
  }

  @override
  Widget build(BuildContext context) {
    var opacity = Opacity(
      opacity: tween.value,
      child: sizedBox,
    );
  }

  @override
  void dispose() {
    controller?.dispose();
    super.dispose();
  }
}

```