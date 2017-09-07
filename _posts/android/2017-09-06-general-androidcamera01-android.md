---
layout: post
title: Android系统中的相机介绍（一）
category: Android
tags: android
description: Android相机介绍
---

## 概述  

### Android系统的起源
#### OS的黎明期
1994年，索尼推出了一款名为MagicjLink的携带型信息终端，这款终端并不仅仅是一款用来记笔记和日程规划的电子帐本，当时已经具备了通信功能还可以收发邮件和搜索信息。MagicLink是索尼公司的产品，但终端的本质是能够在这台终端上运行的OS。这就是由美国的苹果公司的工程师们开发出来的称之为“MagicCap”的OS。这款OS是由苹果的工程师开发，日本的制造商负责终端生产。

苹果的工程师只负责开发了OS，硬件部份是由日本的制造商等外部公司开发出来的。苹果公司还因此设立了“General Magic”这样的一家子公司。这家公司主要负责推进MagicCap的开发和为日本的制造商提供帮助。索尼公司和现在的松下以及三菱等公司以MagicCap终端的开发为名制造出了最初的一款产品即索尼公司的MagicLink。

苹果公司的工程师开发的MagicCap作为“移动OS”是具有划时代意义的。但是MagicCap最终却以失败告终。因为这款终端根本卖不出去。硬件的处理能力和重量以及价格等等在当时都成为了问题。要知道那个时代网络还不是一般人可以使用的。

为MagicCap设想的网络和那个年代即1980年的电脑使用的网络是一样的。而网络和Windows95的爆发性普及却是在那之后的一年才开始的。
#### Android之父--Andy Rubin
实际上在General Magic公司负责MagicCap开发的工程师中有一位叫作Andy Rubin的年轻人。

这名年轻人在General Magic公司解散一年之后创立了一家面向移动终端的OS开发的创业公司。和General Magic公司一样，硬件的开发交由外部公司，本公司集中精力于OS的开发。但是和General Magic公司只向自己的合作公司提供OS不同的是，Andy Rubin的公司免费向其它公司提供OS和APP开发环境。

由Andy Rubin创立的公司正是现在的“Android”。后来这家公司被美国的Google公司收购，而Android这一公司名也就只能作为OS的名称而保留了下来。现在被称之为Android之父的Andy Rubin在公司被收购之后留在了Google公司并且负责Android业务(现在负责机器人的开发)。

#### Android系统本为数码相机系统
Android之父”安迪·鲁宾（Andy Rubin）曾经在日本新经济峰会上透露，Android原本只是一款面向数码相机开发的操作系统，其初衷仅是为了给相机增加“云”的概念，以便在设备接入网络后，能自动备份拍好的照片。

　　在谷歌买下Android后，移动市场开始经历变革。仅过了五个月，谷歌决定转换跑道。

　　“我们当时都认为，数码相机已不再是未来市场上有足够分量的产品。”鲁宾表示，“初如移动市场时，我担心过微软，也担心过塞班系统。但还没有对iPhone表示担心。”

　　谷歌后来推出了“开源手机解决方案”，但继续保留Java为操作系统的核心。为了实现增长，谷歌将Android免费提供，而没有收取任何授权费用。

　　谷歌的决定似乎是正确的，如今该操作系统正向10亿设备安装量的里程碑迈进。

说了这么多，只是为了表示，Android系统对相机应用具备与生俱来的友好，有非常丰富的Api可以使用，如果不是设备大小的限制，真的安装上一枚数码单反的摄像头，它真的能像单反那样拍照，这里指的不是按下快门那么简单，而是指可以像单反那样，调节相机的各种参数，以达到自己想要的效果。其实现在市面上的很多Android手机，出厂自带的相机应用都在手动调节参数这一功能上做了许多工作，很多相机都有手动模式，在光照充足的情况下我们可以使用自动模式拍照，但是在夜间等光照不足的情况下手动模式就显得十分重要了。

# 进入正题
## Android中相机系统架构介绍
先放图，谷歌Android官网对camera的介绍，[谷歌官网地址](https://source.android.com/devices/camera/)。  
![camera结构图](/assets/img/ape_fwk_camera.png)  
这是一张神奇的图，每次看到都能有新的认知。在图中，不同的功能模块都用不同的颜色标识了出来，这些模块都是有对应的Android源码的，有兴趣研究的话可以去下载一套源码读一读。
### Android系统中camera是C/S架构
从宏观的角度来看，Android系统中的Camera采用的是C/S架构，Client端就是各种用到相机的应用（App）或者服务（Service）；Service端是就是系统中的mediaserver进程（上图中有标识出来），不过升级Android7.0之后，谷歌为了提高media和camera的安全性，在架构中将CameraService从MediaService独立了出来，之前的版本中CameraService是包含在MediaService进程中的，所以在7.0之前的版本中经常可以看到MediaService进程挂掉之后Camera也就跟着挂掉了。
应用端打开相机，就是向server请求链接，链接成功后就会创建一个通道，通过这个通道，应用可以向server继续发送各种请求，server端根据请求返回相应的数据给应用，应用拿到数据就可以做自己想做的处理了。
既然是C/S架构那么创建链接就是第一步，上图中我们可以看到，Java层有两个API，都可以实现我们的应用或者服务，也就是说，我们有两种方式创建通道。
## Camera Api1和Api2
首先得说一下这两个东西，因为Camera Api2是谷歌在Android5.0之后新添加的一套api，这套api2相对老的api1来说，接口更加开放，功能更强大，性能也更好，但是同时用起来也稍微复杂一些。

Api1用到的接口绝大多数都是`android.hardware.Camera`；  
Api2用的的接口比较多：  
`android.hardware.camera2.CameraCaptureSession`  
`android.hardware.camera2.CameraCharacteristics`  
`android.hardware.camera2.CameraDevice`  
`android.hardware.camera2.CameraManager`  
`android.hardware.camera2.CaptureFailure`  
`android.hardware.camera2.CaptureRequest`  

以上这些是关于创建链接和传递请求的，不包括server返回的数据相关的处理。返回数据处理的接口也有很多，以后会用到的时候再介绍。
接下来就来看看，应用如何创建这个通道。
## 打开相机
前边已经介绍过，上层可以用的接口有两套，Api1和Api2。
### Api1
Api1的接口调用起来相对比较简单，打开相机只需要一行代码就可以了：  
```java
Camera camera = Camera.open(0);
```
这是Android的原生接口，这里的Camera类型就是android.hardware.Camera。  

这里的open()是一个静态(static)方法，直接用类名调用。方法可以接收一个int类型的参数，也可以不传参数。这个参数就是指定想要打开的相机的id，一般来说现在的手机上都有前摄和后摄这至少两个摄相头，而且一般情况下后摄的的camera id是0，前摄的camera id是1。前面说了这个方法也可以不传参数，在不传参数的情况下，会默认去打开后摄。至于返回值camera，如果打开成功的话会返回这个Camera类行的实例(这个实例就是指向C/S架构通道Client端的引用，之后直接对它进行操作就可以将命令发送到Server端)，如果打开失败的话返回值就是null，所以可以通过返回值camera是否为null来判断本次打开相机是否成功。还有一个接口可以帮助我们在请求打开相机之前就判断系统有几个摄像头
```java  
int number = Camera.getNumberOfCameras();
```
这个可以做什么？也许对打开相机作用不大，不过你可以根据这个值来判断一下你的应用是否需要切换摄像头这个图标，如果系统只有一个摄像头的话，就隐藏掉切换摄像头的图标。  

怎么样，简单吧！  
![想不到吧](/assets/img/surprise.jpg)  
### Api2
用api2来获得我们的client端的时候稍微负责一些，先来贴一些代码：  
```java
CameraManager mCameraManager = (CameraManager) mActivity.getSystemService(Context.CAMERA_SERVICE);
CameraDevice mCamera;
CameraDevice.StateCallback mCameraDeviceStateCallback =
            new CameraDevice.StateCallback() {
                @Override
                public void onOpened(CameraDevice camera) {
                    Log.i(TAG, "onOpened CameraDevice camera=" + camera);
                    mCamera = camera;
                }

                @Override
                public void onDisconnected(CameraDevice camera) {
                    Log.w(TAG, "Camera device '" + camera + "' was disconnected");
                }

                @Override
                public void onError(CameraDevice camera, int error) {
                    Log.e(TAG, "Camera device encountered error code '" + error + '\'');
                }
            };
mCameraManager.openCamera(mCameraId, mCameraDeviceStateCallback, mOptionHandler);
```
然后来解释一下，这段代码什么意思。  
首先，用Api2打开相机的接口是：
```java
mCameraManager.openCamera(mCameraId,  //String类型，要打开的camera的id
 mCameraDeviceStateCallback,  //CameraDevice.StateCallback类型，server端camera有什么状态变化就会回调这个接口通知client端
 mOptionHandler);  //一个Handler，可以自己创建一个线程获取handler，也可以直接用主线程的handler，也可以传null，传null的话它会自动获取住线程的handler
```
这个接口需要3个参数，一个id，一个stateCallBack，一个handler，所以我们要明确id，创建callback，然后再给个handler。  
然后相机打没打开，就在stateCallBack的回调中了，回调方法的名字谷歌取的也很简洁明了，让人一看就明白是怎么回事。
如果相机成功打开了，我们就可以获得一个CameraDevice实例，这个实例就是我们client端的通道接口，通过它可以向server端发送命令，也可以接收server端传来的数据。  

相机打开了，我们的工作才算是开始了。
