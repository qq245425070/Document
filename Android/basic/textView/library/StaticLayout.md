### StaticLayout  
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