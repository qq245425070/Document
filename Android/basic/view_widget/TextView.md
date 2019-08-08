### 测量文本宽度-长度  
在5.5英寸 1080P的手机上，  
中文标点符号 都是48px；  
汉字都是48px；  
英文' 是8px；  
英文, 是9px；  
空格  是12px；  
英文. 是13px；  
英文a是26px；    
英文A是32px；  
```
public class EllipsizeTextView extends android.support.v7.widget.AppCompatTextView {
    private String endCharSeq = "...";
    private int maxLine = 1;
    private int realWidth = -1;
    private int maxRealWidth = -1;
    
    public EllipsizeTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
        maxLine = getMaxLines();
    }
    
    @SuppressLint("DrawAllocation")
    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        if (realWidth < 0) {
            realWidth = getWidth() - getPaddingLeft() - getPaddingRight();
        }
        if (maxRealWidth < 0) {
            maxRealWidth = realWidth * maxLine;
        }
        CharSequence mCharSeq = getText();
        int length = mCharSeq.length();
        int widthCount = 0;
        int lastIndex = length;
        for (int i = 0; i < length; i++) {
            CharSequence subSeq = mCharSeq.subSequence(i, i + 1);
            widthCount += Layout.getDesiredWidth(subSeq, 0, 1, getPaint());
            LogTrack.i(subSeq + "  " + widthCount);
            if (widthCount >= maxRealWidth) {
                lastIndex = i;
                LogTrack.w(subSeq + "  " + widthCount + "  " + maxLine + "  " + realWidth);
                break;
            }
        }
        setText(mCharSeq.subSequence(0, lastIndex-1));
    }
}
```  
### 设置删除线  

```
/**
 * 设置删除线
 */
public static <V extends TextView> void throughLine(V view) {
    if (view == null) {
        return;
    }
    view.getPaint().setFlags(Paint.ANTI_ALIAS_FLAG | Paint.STRIKE_THRU_TEXT_FLAG);
}

/**
 * 设置下划线
 */
public static <V extends TextView> void underLine(V view) {
    if (view == null) {
        return;
    }
    view.getPaint().setFlags(Paint.ANTI_ALIAS_FLAG | Paint.UNDERLINE_TEXT_FLAG);
}


/**
 * 文本加粗
 */
public static <V extends TextView> void fakeBold(V view) {
    if (view == null) {
        return;
    }
    view.getPaint().setFlags(Paint.ANTI_ALIAS_FLAG | Paint.FAKE_BOLD_TEXT_FLAG);
}

```
### StaticLayout  
```
1.字符串子资源  
2 .画笔对象  
3.layout的宽度，字符串超出宽度时自动换行。  
4.layout的样式，有ALIGN_CENTER， ALIGN_NORMAL， ALIGN_OPPOSITE  三种。  
5.相对行间距，相对字体大小，1.5f表示行间距为1.5倍的字体高度。  
6.相对行间距，0表示0个像素。  
实际行间距等于这两者的和。  
7.还不知道是什么意思，参数名是boolean includepad。  
需要指出的是这个layout是默认画在Canvas的(0,0)点的，如果需要调整位置只能在draw之前移Canvas的起始坐标  
canvas.translate(x,y);   
```  
StaticLayout 静态Label，如TextView  
DynamicLayout 编辑框，如EditText  

android:focusable="true"  
android:focusableInTouchMode="true"  
第一次点击 onClick 不会响应，之后再点击，onClick 都会响应；  


### 参考  
http://blog.csdn.net/ysh06201418/article/details/46439561  
http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/0915/1682.html  
http://blog.csdn.net/kongbaidepao/article/details/48247217  
http://www.cnblogs.com/July-and-Sky/p/5527883.html  
https://www.zhihu.com/question/23768161/answer/82424947  
