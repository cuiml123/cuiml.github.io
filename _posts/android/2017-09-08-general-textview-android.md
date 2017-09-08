---
layout: post
title: Android的TextView
category: Android
tags: android
description: Android的Textview的几点使用
---

## Textview

TextView是android的一种原生控件，顾名思义，它是主要用来显示文字的控件，本文记录几种TextView的效果实现方法。

### 阴影
给文字设置阴影的方法：
```java
        TextView tv = (TextView) findViewById(R.id.tx);
        tv.setShadowLayer(3f, 3f, 3f, Color.BLUE);
```
可以在文字的边缘加上阴影效果
效果图  
![setShadowLayer](/assets/img/shadowlayer.png)  
接口比较简单，setShadowLayer()方法需要四个参数
```java
setShadowLayer(
float radius, //阴影的大小，也就是宽度
float dx, //x轴方向的偏移量
float dy, //y轴方向的偏移量
int color) //阴影的颜色
```
换个参数对比一下，代码：
```java
        TextView tv = (TextView) findViewById(R.id.tx);
        tv.setShadowLayer(10f, 0f, 0f, Color.BLUE);
```
效果图  
![shadowlayer2](/assets/img/shadowlayer2.png)  


当然也可以在布局文件中配置相应的属性
```xml
    <TextView
        android:id="@+id/tx"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:textSize="50sp"
        android:textColor="@android:color/white"
        android:shadowColor="@android:color/black"
        android:shadowDx="3"
        android:shadowDy="3"
        android:shadowRadius="3"
        android:text="ShadowLayer"/>
```
### 跑马灯效果
实现TextView中的文字的滚动效果：
java代码实现：   
```java
        TextView tv = (TextView) findViewById(R.id.tx);
        tv.setSingleLine();
        tv.setFocusable(true);
        tv.setFocusableInTouchMode(true);
        tv.setEllipsize(TextUtils.TruncateAt.MARQUEE);
        tv.setMarqueeRepeatLimit(-1);
```
xml布局实现：  
```xml
    <TextView
        android:id="@+id/tx"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:textSize="50sp"
        android:textColor="@android:color/white"
        android:singleLine="true"
        android:focusable="true"
        android:focusableInTouchMode="true"
        android:ellipsize="marquee"
        android:marqueeRepeatLimit="marquee_forever"
        android:text="TextUtils.TruncateAt.MARQUEE"/>
```
4个注意点
- 文字单行显示，文字长度比TextView长
- 设置ellipsize为marquee
- 设置marqueeRepeatLimit，-1为forever
- TextView要有焦点focus  
![marquee](/assets/img/marquee.gif)  

### 颜色渐变
android中实现颜色渐变可以用LinearGradient，创建一个想要的渐变效果LinearGradient，然后将这个渐变效果设置给画笔Paint，Paint.setShader(LinearGradient)，就可以实现颜色的渐变。  

如何获取画笔Paint呢，一种当然是自定义View，可以创建自己的画笔Paint并重写onDraw()方法；另一种，比如本文的主角TextView，它有getPaint()方法可以拿到画笔Paint。  

简单的实现代码：
```java
    TextView tv = (TextView) findViewById(R.id.tx);
    //tv.setSingleLine();
    LinearGradient mLinearGradient = new LinearGradient(0, 0, tv.getMeasuredWidth() / 2, tv.getMeasuredHeight(),
            Color.RED, Color.WHITE, Shader.TileMode.CLAMP);
    tv.getPaint().setShader(mLinearGradient);
```
效果图  
![gradient](/assets/img/gradient1.png)

LinearGradient的创建：
LinearGradient有两种构造方法  
- 1
```java
LinearGradient(float x0, //线性起点的x坐标
  float y0, //线性起点的y坐标
  float x1, //线性终点的x坐标
  float y1, //线性终点的y坐标
  int colors[], //渐变效果的颜色的组合
  float positions[], //前面的颜色组合中的各颜色在渐变中占据的位置（比重），如果为空，则表示上述颜色的集合在渐变中均匀出现
  TileMode tile)  //渲染器平铺的模式
```
渲染器平铺的模式，一共有三种  
-CLAMP  
边缘拉伸  
-REPEAT  
在水平和垂直两个方向上重复，相邻图像没有间隙  
-MIRROR  
以镜像的方式在水平和垂直两个方向上重复，相邻图像有间隙  
- 2
```java
public LinearGradient(float x0, float y0, float x1, float y1,
  int color0, //渐变起始颜色
  int color1, //渐变终止颜色
  TileMode tile)  
```
其他参数同上  

前面的例子就是使用的第二种，颜色变化比较简单一些。  
用第二种构造方法试试：
```java
    TextView tv = (TextView) findViewById(R.id.tx);
    //tv.setSingleLine();
    LinearGradient mLinearGradient = new LinearGradient(0, 0, tv.getMeasuredWidth() / 4, tv.getMeasuredHeight() / 4,
            new int[]{Color.RED, Color.WHITE, Color.BLUE, Color.BLACK},
            new float[]{0f, 0.3f, 0.6f, 1.0f},
            Shader.TileMode.REPEAT);
    tv.getPaint().setShader(mLinearGradient);
```
效果  
![gradient2](/assets/img/gradient2.png)  

需要注意，getMeasuredHeight方法要在onResume完成之后才有效；如果存在横向的渐变，就不能设置tv.setSingleLine()，那样会使得渐变失效；只有纵向的渐变的话可以添加tv.setSingleLine()，然后让文字实现跑马灯效果动起来。

## iconfont的使用
iconfont的简单使用
iconfont在Android中的使用官网已经做了非常详细介绍：
[iconfont的使用方法](http://www.iconfont.cn/help/detail?helptype=code)

- 1、下载要用的iconfont资源；
- 2、将其中的 iconfont.svg 和 iconfont.ttf 文件添加到项目中的 assets 文件夹中；
- 3、在目录中打开demo.html，找到图标相对应的 HTML 实体字符码；
- 4、在string.xml中配置字符串引用对应的实体字符码；
- 5、给TextView设置Typeface；

代码示例：  
布局
```xml
    <TextView
        android:id="@+id/tx"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:textSize="40sp"
        android:text="@string/hello"/>
```
string文件添加
```xml
    <string name="hello">金牛座 &#xe6bf; 双鱼座 &#xe6c6; 射手座 &#xe6cb;</string>
```
给TextView设置文字
```java
    TextView tv = (TextView) findViewById(R.id.tx);
    Typeface iconfont = Typeface.createFromAsset(getAssets(), "iconfont.ttf");
    tv.setTypeface(iconfont);
    LinearGradient mLinearGradient = new LinearGradient(0, 0, 0, tv.getMeasuredHeight() / 4,
            Color.RED, Color.BLUE,
            Shader.TileMode.MIRROR);
    tv.getPaint().setShader(mLinearGradient);
```
效果  
![iconfont](/assets/img/iconfont.png)  
