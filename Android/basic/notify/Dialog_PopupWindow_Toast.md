### Dialog、PopupWindow、Toast  

[Dialog 和 PopupWindow 的比较](Dialog_PopupWindow_Compare.md)  
[处理PopupWindow弹出高度和位置](pop_window_height.md)  

### 常用处理  
popupWindow 一定要在 onPause 执行 dismiss, 否则有可能会出现异常;  
#### 底部弹出 popupWindow  
背景透明  
```
    @OnClick(R2.id.title)
    void clickQuiz() {
        quizPopupWindow.showAtLocation(getWindow().getDecorView(), quizList, quizsBean);
    }

MyPopupWindow.showAtLocation(){
    WindowManager.LayoutParams attributes = activity.getWindow().getAttributes();
    attributes.alpha = 0.5f;
    activity.getWindow().setAttributes(attributes);
    showAtLocation(parent, Gravity.BOTTOM, 0, -10);
    contentView.post(() -> showAtLocation(parent, Gravity.BOTTOM, 0, -contentView.getHeight()));
}
```
https://github.com/Dovar66/DToast  
