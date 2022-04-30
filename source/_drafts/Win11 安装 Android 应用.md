# Win11 安装 Android 应用

在去年10月21日，微软宣布Win11上支持运行Android 应用程序的安卓子系统·`Windows subsystem for Android`第一个预览版现已提供给**美区Beta**频道上的用户，现在美区的所有Beta预览频道用户都可以通过此应用和Amazon AppStore（亚马逊应用商店）在Windows11上运行Android应用。

微软还表示他们跟亚马逊合作，为Windows Beta预览频道上的用户推出了50款安卓App，包括Kindle、王国纪元。

![img](https://pic2.zhimg.com/80/v2-27415856a44e9379b758457fa17a9d7d_1440w.jpg)

看到这个消息，我马上给电脑安装了Windows Subsystem For Android并通过一些**手段**用它成功的在Windows11上运行了**酷安、支付宝、微信**等国内的App。今天就跟大家分享一下自己是怎么给电脑安装上Android软件的吧。

![img](https://pic1.zhimg.com/80/v2-2433c789c8d73a0d59d63ce89b08ff90_1440w.jpg)

# 准备工作和安装要求

**1.**电脑硬件**必须**要满足Windows11的最低硬件要求，且电脑需要安装Windows11

**2.**系统必须处于Beta预览频道

![img](https://pic4.zhimg.com/80/v2-1c79e4d25d09c1a1f3aeaadd837b21e7_1440w.jpg)

**3.**要保证电脑**已启用虚拟化**：

如果电脑没有开启虚拟化的话，大家可以到自己电脑的官网查看打开虚拟化方法

![img](https://pic4.zhimg.com/80/v2-49502bce172502db76fcefcf32213877_1440w.jpg)

**4.**要保证电脑上没有安装其他安卓模拟器，也要保证Windows Sandbox（Windows沙盒）处于关闭状态，而**虚拟机平台要处于开启状态**。如果已经打开了Windows沙盒的话，需要关闭它并重启电脑。打开控制面板，依次点击【程序】-【启用或关闭Windows功能】-取消勾选Windows沙盒后重启电脑就OK了。虚拟机平台也可以在此页面打开，打开后别忘了重启电脑：

![img](https://pic1.zhimg.com/80/v2-0a54a4f73b62b025c647c88033897704_1440w.jpg)

**5.**将区域切换为**美国**

打开设置，依次点击【时间和语言】-【语言和区域】，把国家切换为美国：

![img](https://pic4.zhimg.com/80/v2-36a275162edc3ed400247ef1557203eb_1440w.jpg)

# 安装安卓子系统、亚马逊商店

目前安装安卓子系统Windows Subsystem for Android的方法有两种：

1. 直接安装Windows Subsystem for Android
2. 先安装Amazon AppStore，系统检测到用户安装Amazon AppStore后会自动安装安卓子系统Windows Subsystem for Android。



**直接安装安卓子系统**

安卓子系统下载地址（大小在1-1.5GB左右，需要等几分钟）：

https://www.microsoft.com/store/productId/9P3395VX91NR

![img](https://pic2.zhimg.com/80/v2-2b8d7316e9f2d71f75c8a1bce8fea3a1_1440w.jpg)

**通过Amazon AppStore间接安装安卓子系统**

Amazon AppStore下载地址：

http://aka.ms/AmazonAppstore

![img](https://pic3.zhimg.com/80/v2-c05dc09e2cf5b1a7323c58bcf7720f8a_1440w.jpg)

安装完安卓子系统和Amazon AppStore后，在开始菜单上可以看到它们：

![img](https://pic4.zhimg.com/80/v2-9ccca494f55b78a9ec416968b8c383eb_1440w.jpg)

安卓子系统界面如图所示：

![img](https://pic2.zhimg.com/80/v2-59e03f6bdd921df26c7b57615e86e615_1440w.jpg)

**注意：建议大家开启它的开发者模式和可选诊断数据开关，这样可以更好的用其他软件给它安装安卓App**

# 通过Amazon AppStore安装应用

微软表示如果想在此安卓子系统上安装软件，需要通过Amazon AppStore进行安装。

也就是说，我们还得注册一个Amazon AppStore账号...但是我折腾了一会儿成功注册了Amazon AppStore账号后发现，国内用户仍然无法正常使用Amazon AppStore下载软件...

![img](https://pic3.zhimg.com/80/v2-d5f422e939d8ec10855dcb47ab5b8ffa_1440w.jpg)

所以，如果大家有**美区**Amazon AppStore账号的话，现在可以从Amazon AppStore给电脑安装安卓App了。

如果没有**美区**Amazon AppStore账号的话，那就请跟着我进行下一步操作。用不了Amazon AppStore的话，那我们就用其他**手段**给它安装上应用...

# 通过第三方工具安装安卓应用

接下来给大家介绍一下现在要用到的软件：**Apk Installer**

![img](https://pic4.zhimg.com/80/v2-94a39c8e505242b2a307a3c616d6cdef_1440w.jpg)

Apk Installer可以检测连接在电脑上的安卓设备或电脑上的安卓系统并给它安装软件。

![img](https://pic3.zhimg.com/80/v2-6438224f0fc878c76f7c2b3299e8bbe6_1440w.jpg)

现在有了Apk Installer，我们可以给已安装安卓子系统的电脑安装任何一种安卓App...

**那么，你的电脑准备好了吗？我们要开始给它安装微信了！**

我们要先从微信官网下载微信安卓版（Apk)。微信下好了之后双击运行下好的Apk Installer。

将微信安装包拖到Apk Installer里面，勾选为微信后面的小方框后点击【Install】就OK了：

![img](https://pic2.zhimg.com/80/v2-3737640cf2b6ff28ba708e160b796fe9_1440w.jpg)

不到几秒就会安装成功：

![img](https://pic4.zhimg.com/80/v2-c55c9fb8123e45937359ea4c6e67d16b_1440w.jpg)

此时打开开始菜单就可以看到微信了

运行效果如下（安卓子系统第一次启动时需要几秒钟的加载时间）：

我用同样的方法给电脑安装了支付宝、酷安等其他App：

![img](https://pic4.zhimg.com/80/v2-e710c303703a46c6c68c964532ceaa9f_1440w.jpg)



![img](https://pic1.zhimg.com/80/v2-d31f57afe10ce8b4c5502880c746f830_1440w.jpg)

运行微信等App时，安卓子系统CPU占用率和内存占用率如下：

![img](https://pic2.zhimg.com/80/v2-67363d59b3a79136a49c927552d40ce1_1440w.jpg)

电脑内存是8GB的用户要谨慎使用哦。

如果运行安卓App时出现了一些问题，如程序卡死、App闪退等等，那么大家可以从开始菜单打开Windows Subsystem for Android程序，点击【关闭适用于Android的子系统】后再重新打开安卓App就可以了。这个操作类似于Windows上的“重启大法”：

![img](https://pic4.zhimg.com/80/v2-1b782a9a3a09433d4a58ea63aa8f7567_1440w.jpg)



![img](https://pic2.zhimg.com/80/v2-a4ceeeb3c58d360623378c7328a71f31_1440w.jpg)



# 总结

**存在的问题：**

目前安卓子系统Windows Subsystem for Android还处在测试阶段，所以使用时偶尔会出现应用闪退、窗口闪烁、应用无法联网、不能调用电脑摄像头等问题。

具体问题可见[官网](https://docs.microsoft.com/en-us/windows/android/wsa/)。

**特点：**

- 给它安装的安卓App会直接出现在开始菜单上，我们可以在开始菜单上对它们进行一些操作，如卸载、固定到任务栏等等，非常的方便。
- Windows系统和安卓子系统可以相互读取剪切板。
- 安卓子系统中的通知会直接出现在Windows通知中心中，管理通知非常方便。
- 安卓子系统支持Windows的一些快捷键。

![img](https://pic4.zhimg.com/80/v2-f78b130c238858e157d589f6e49fcf1b_1440w.jpg)

**补充一下：**

**用Apk installer时不仅建议大家打开开发人员模式，还建议先打开一次Amazon AppStore再用Apk installer安装软件。**

**因为最近发现有时候安卓子系统虚拟机进程没有运行时，Apk installer发现不了它，导致Apk installer无法正常安装软件。使用前打开一次Amazon App Store的话，虚拟机进程就会激活，这时再用Apk Installer就可以安装软件了。**

**怎么查看虚拟机进程是否在运行？**

**方法1.打开任务管理器看到vmmem WSA就说明虚拟机进程在运行。**

**方法2. 打开windows subsystem for android可以看到一个连接Adb的端口号就说明虚拟机进程正在运行，此时使用Apk installer就可以安装软件**

![img](https://pic1.zhimg.com/80/v2-49f624c441f66f438537c3ed32f48b24_1440w.jpg)