# Canvas之绘制基本形状


### 简要介绍Paint

看了上面这么多，相信有一部分人会产生一个疑问，如果我想绘制一个圆，只要边不要里面的颜色怎么办？

很简单，绘制的**基本形状由Canvas确定**，但绘制出来的**颜色,具体效果则由Paint确定**。

如果你注意到了的话，在一开始我们设置画笔样式的时候是这样的：

``` java
  mPaint.setStyle(Paint.Style.FILL);  //设置画笔模式为填充
```

为了展示方便，容易看出效果，之前使用的模式一直为填充模式，实际上画笔有三种模式，如下：

``` java
STROKE                //描边
FILL                  //填充
FILL_AND_STROKE       //描边加填充
```

为了区分三者效果我们做如下实验：

```
        Paint paint = new Paint();
        paint.setColor(Color.BLUE);
        paint.setStrokeWidth(40);     //为了实验效果明显，特地设置描边宽度非常大

        // 描边
        paint.setStyle(Paint.Style.STROKE);
        canvas.drawCircle(200,200,100,paint);

        // 填充
        paint.setStyle(Paint.Style.FILL);
        canvas.drawCircle(200,500,100,paint);

        // 描边加填充
        paint.setStyle(Paint.Style.FILL_AND_STROKE);
        canvas.drawCircle(200, 800, 100, paint);
```

<img src="http://ww4.sinaimg.cn/large/005Xtdi2jw1f274g6lxbpj30u01hcq3n.jpg" width = "300" /> 

一图胜千言，通过以上实验我们可以比较明显的看出三种模式的区别，如果只需要边缘不需要填充内容的话只需要设置模式为描边(STROKE)即可。