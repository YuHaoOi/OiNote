
# 自定义View分类与流程

## 前言


自定义View绘制流程函数调用链(简化版)

![](https://github.com/YuHaoOi/OiNote/blob/master/Android/pic/cusomview_life.png?raw=true)


*******

##  几个重要的函数


### 1.测量View大小(onMeasure)


测量View大小使用的是onMeasure函数，我们可以从onMeasure的两个参数中取出宽高的相关数据：
``` java
   @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int widthsize = MeasureSpec.getSize(widthMeasureSpec);      //取出宽度的确切数值
        int widthmode = MeasureSpec.getMode(widthMeasureSpec);      //取出宽度的测量模式
        
        int heightsize = MeasureSpec.getSize(heightMeasureSpec);    //取出高度的确切数值
        int heightmode = MeasureSpec.getMode(heightMeasureSpec);    //取出高度的测量模式
    }
```
从上面可以看出 onMeasure 函数中有 widthMeasureSpec 和 heightMeasureSpec 这两个 int 类型的参数， 毫无疑问他们是和宽高相关的， **但它们其实不是宽和高， 而是由宽、高和各自方向上对应的测量模式来合成的一个值：**

**测量模式一共有三种， 被定义在 Android 中的 View 类的一个内部类View.MeasureSpec中：**

| 模式          | 二进制数值 | 描述                                     |
| ----------- | :---: | -------------------------------------- |
| UNSPECIFIED |  00   | 默认值，父控件没有给子view任何限制，子View可以设置为任意大小。    |
| EXACTLY     |  01   | 表示父控件已经确切的指定了子View的大小。                 |
| AT_MOST     |  10   | 表示子View具体大小没有尺寸限制，但是存在上限，上限一般为父View大小。 |

#### 注意：
**如果对View的宽高进行修改了，不要调用*super.onMeasure(widthMeasureSpec,heightMeasureSpec);*要调用*setMeasuredDimension(widthsize,heightsize);* 这个函数。**

### 2.确定View大小(onSizeChanged)
  这个函数在视图大小发生改变时调用。

**Q: 在测量完View并使用setMeasuredDimension函数之后View的大小基本上已经确定了，那么为什么还要再次确定View的大小呢？**

**A: 这是因为View的大小不仅由View本身控制，而且受父控件的影响，所以我们在确定View大小的时候最好使用系统提供的onSizeChanged回调函数。**

onSizeChanged如下：
``` java
    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
    }
```
可以看出，它又四个参数，分别为 宽度，高度，上一次宽度，上一次高度。

这个函数比较简单，我们只需关注 宽度(w), 高度(h) 即可，这两个参数就是View最终的大小。


### 3.确定子View布局位置(onLayout)

**确定布局的函数是onLayout，它用于确定子View的位置，在自定义ViewGroup中会用到，他调用的是子View的layout函数。**

  在自定义ViewGroup中，onLayout一般是循环取出子View，然后经过计算得出各个子View位置的坐标值，然后用以下函数设置子View位置。

``` java
  child.layout(l, t, r, b);
```
四个参数分别为：

| 名称   | 说明                | 对应的函数        |
| ---- | ----------------- | ------------ |
| l    | View左侧距父View左侧的距离 | getLeft();   |
| t    | View顶部距父View顶部的距离 | getTop();    |
| r    | View右侧距父View左侧的距离 | getRight();  |
| b    | View底部距父View顶部的距离 | getBottom(); |