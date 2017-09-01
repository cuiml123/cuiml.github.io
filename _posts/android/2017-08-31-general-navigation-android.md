---
layout: post
title: Android导航栏、状态栏，全屏显示
category: Android
tags: android
description: Android导航栏、状态栏相关的知识
---

## 硬件配置
Android手机都是可以显示导航栏的，不过有实体键的手机没有必要显示底部导航栏，于是就配置为不显示导航栏了。
android手机中，系统是否可以显示导航栏是由系统配置决定的：
可以用`adb shell`命令进到手机中进行查看，配置文件的位置`system/build.prop`  
可以使用以下命令查看：
```bash
$ cat system/build.prop | grep qemu
qemu.hw.mainkeys=1
```
或者这个命令，大同小异：
```bash
$ getprop system/build.prop | grep qemu
qemu.hw.mainkeys=1
```
修改`qemu.hw.mainkeys=1`的值为0，`qemu.hw.mainkeys=0`,然后重启手机，就可以使系统设置中显示出导航栏设置项，如果它本来就是0,那么你的手机上应该就显示导航栏了。以上操作的前提是手机有root权限&nbsp;：）  
## 应用中动态设置导航栏的显示状态
在Android4.0及以后版本中，可通过以下方法隐藏NavigationBar
```java
    View decorView = getWindow().getDecorView();  
    // Hide both the navigation bar and the status bar.  
    // SYSTEM_UI_FLAG_FULLSCREEN is only available on Android 4.1 and higher, but as  
    // a general rule, you should design your app to hide the status bar whenever you  
    // hide the navigation bar.  
    int uiOptions = View.SYSTEM_UI_FLAG_HIDE_NAVIGATION  
                  | View.SYSTEM_UI_FLAG_FULLSCREEN;  
    decorView.setSystemUiVisibility(uiOptions);  
```

### 其他`setSystemUiVisibility(int)`相关的参数：

- `SYSTEM_UI_FLAG_VISIBLE`：显示状态栏和导航栏。

- `SYSTEM_UI_FLAG_LOW_PROFILE`：此模式下，状态栏的图标可能是暗的。

- `SYSTEM_UI_FLAG_HIDE_NAVIGATION`：底部全屏，隐藏导航栏。

- `SYSTEM_UI_FLAG_FULLSCREEN`：顶部全屏，隐藏状态栏。

- `SYSTEM_UI_FLAG_LAYOUT_STABLE`：布局大小会固定，不随状态栏和导航栏移动。

- `SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION`：全屏，沉到状态栏和导航栏后面。

- `SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN`：顶部全屏，沉到状态栏后面。

- `SYSTEM_UI_FLAG_IMMERSIVE`：沉浸式。

- `SYSTEM_UI_FLAG_IMMERSIVE_STICKY`：粘性沉浸式。

### 更多
#### `SYSTEM_UI_FLAG_FULLSCREEN`与`SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN`区别：
1. 单独使用`SYSTEM_UI_FLAG_FULLSCREEN`时，应用会全屏状态栏会隐藏，但是，当下滑显示状态栏后，状态栏会再次显示，伴随着应用的顶部会下移。  
![SYSTEM_UI_FLAG_FULLSCREEN单独使用](/assets/img/full.gif)  
请不要与`SYSTEM_UI_FLAG_LAYOUT_STABLE`单独搭配使用，会导致顶部留白，然后下滑后显示状态栏。
2. 单独使用`SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN`时，应用全屏，但是顶部状态栏不会隐藏。  
![SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN单独使用](/assets/img/layout_full_single.gif)
3. 两者一起使用，进入应用后会全屏并隐藏顶部状态栏，用户从顶部下滑呼出状态栏后，应用的布局不会变化，但是状态栏也不会消失。  
![两者同时使用](/assets/img/layoutfull_full.gif)  
现在可以总结一下：`SYSTEM_UI_FLAG_FULLSCREEN`是会让状态栏消失的顶部全屏，同时状态栏也可以让它消失；`SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN`是让应用顶部全屏，不管状态栏怎么样。  

#### `SYSTEM_UI_FLAG_HIDE_NAVIGATION`与`SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION`的区别：  
1. 单独使用`SYSTEM_UI_FLAG_HIDE_NAVIGATION`时，导航栏会隐藏，但是点击应用任意位置，导航栏会出现，应用底部会上移。同样的，请不要与`SYSTEM_UI_FLAG_LAYOUT_STABLE`单独搭配使用，会导致低部留白，点击后显示导航栏。  
![SYSTEM_UI_FLAG_HIDE_NAVIGATION单独使用](/assets/img/hidenav.gif)  
2. 单独使用`SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION`时，应用会全屏，与`SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN`的全屏类似，顶部会沉到状态栏后面，另外底部也会沉到导航栏后面。  
![SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION单独使用](/assets/img/layout_hidenav_single.gif)    
3. 两者一起使用时，当然，效果也会叠加，刚设置时，应用全屏，顶部沉在状态栏后面，底部导航栏隐藏，点击应用任意位置之后，底部导航栏出现，遮住应用底部。  
![两者同时使用](/assets/img/layouthidenav_hidenav.gif)  
总结：与fullscreen标签类似，`SYSTEM_UI_FLAG_HIDE_NAVIGATION`可以使底部导航栏隐藏，同时底部导航栏也可以让他消失，但是不会影响顶部；`SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION`是让应用的布局大小为全屏，不管状态栏和导航栏的状态怎么样，会让顶部也达到全屏状态。

##### 再再总结：
以上的实验中我们可以看到：
1. `SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION`就可以让应用的布局为全屏尺寸，忽略掉状态栏和导航栏；
2. `SYSTEM_UI_FLAG_FULLSCREEN`就可以让状态栏暂时隐藏；
3. `SYSTEM_UI_FLAG_HIDE_NAVIGATION`就可以让底部导航栏暂时隐藏。

那么我们要`SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN`做什么？它的作用是让顶部布局达到全屏忽略状态栏状态，这作用与`SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION`有重复。对了，如果要设置为全屏顶部和底部都达到全屏状态，那么一个`SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION`就够了。

#### `SYSTEM_UI_FLAG_IMMERSIVE`与`SYSTEM_UI_FLAG_IMMERSIVE_STICKY`的区别：  
前面介绍了几个标签，可以达到全屏显示的效果，既然要全屏就要隐藏状态栏和导航栏，就需要设置`SYSTEM_UI_FLAG_HIDE_NAVIGATION`和`SYSTEM_UI_FLAG_FULLSCREEN`，但是有一个问题，设置完标签之后，界面可以暂时全屏，但是只要碰触到应用的界面，导航栏和状态栏就会又显示出来：  
![fullscreen加hidenav](/assets/img/full_hidenav.gif)  
为了解决这个问题，有了IMMERSIVE（沉浸式）标签
1. `SYSTEM_UI_FLAG_IMMERSIVE`，添加这个标签之后，全屏状态下点击应用，不会再触发导航栏和状态栏，但是顶部下拉是可以触发的，不然没有导航栏，没法正常使用手机。导航栏和状态栏出现后，不会自动消失，除非再次设置全屏标签。  
![fullscreen+hidenav+immersive](/assets/img/full_hidenav_nosticky.gif)  
2. `SYSTEM_UI_FLAG_IMMERSIVE_STICKY`，使用这个标签之后，基本作用与`SYSTEM_UI_FLAG_IMMERSIVE`相同，不同的是，导航栏和状态栏出现之后过一段时间会自动隐藏，点击应用界面会加快导航栏和状态栏的隐藏。  
![full+hidenav+imsticky](/assets/img/full_hidenav_imsticky.gif)  
总结：`SYSTEM_UI_FLAG_IMMERSIVE`和`SYSTEM_UI_FLAG_IMMERSIVE_STICKY`都可以防止触摸事件呼出状态栏和导航栏，不同的是`SYSTEM_UI_FLAG_IMMERSIVE_STICKY`可以让导航栏和状态栏在被主动呼出后再次自动隐藏。另外，在最后的两个测试中，都只设置了`SYSTEM_UI_FLAG_HIDE_NAVIGATION`和`SYSTEM_UI_FLAG_FULLSCREEN`标签，而没有设置LAYOUT相关的，然而结果可以发现`SYSTEM_UI_FLAG_IMMERSIVE_STICKY`的加入使得呼出状态栏和导航栏时，我们的应用布局并没有被向内压缩，而使用`SYSTEM_UI_FLAG_IMMERSIVE`标签时，呼出导航栏和状态栏时应用布局跟着向内变小了。
