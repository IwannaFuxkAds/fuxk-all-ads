# 准备
- 访问 [Apkpure](https://apkpure.com/) （需要梯子）搜索 USB Camera 获取最新版本
	- 使用MT管理器的“去除签名校验”功能
- MT管理器（需要VIP功能，付费困难请下载破解版）
- adb工具
- 【可选】QtScrcpy —— 安卓投屏到电脑操控的软件，自带adb工具

# 开始

## 观察有哪些广告
- 固定在屏幕中上方的广告
- 弹出的SnackBar广告（提示购买Pro，只会出现一次，暂且不管）
- 断开USB之后的弹窗广告

## 方法一：直接删除谷歌广告服务的代码
然后通过报错信息跟踪产生错误的代码，并删除

### 1. 寻找广告线索
- 如果是弹出整个页面的广告，可以查看Activity，详见 [[#其中包含了当前的Activity以及pid|获取当前Activity]]
- 固定在页面的广告就查看Activity的布局信息

### 2. 详细步骤
通过[[#其中包含了当前的Activity以及pid|获取当前Activity]]可得：
```
ACTIVITY com.shenyaocn.android.usbcamera/com.google.android.gms.ads.AdActivity
```
找到对应广告的代码包：`com.google.android.gms.ads`
用MT管理器打开 `classes.dex`，删除 `/com/google.android/gms/ads` 文件夹
保存，运行，得到报错信息

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231122085416763.png]]

找到 `com.shenyaocn.android.usbcamera.BaseAppActivity.I` 方法

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231122085555866.png]]

看不懂 smali 的话就右上角——转换成java

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231122085721313.png]]

因为进行了 `try-catch` ，所以直接删也不会产生影响 
![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231122085902706.png|200]]
（删除了方法体之后）

再次打包运行，

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231122090058960.png]]

找到 `com.shenyaocn.android.usbcamera.BaseAppActivity.G` 方法

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231122090202411.png]]

是获取 `AdView` 的，和广告相关的东西，这里让他直接返回null就行

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231122090305131.png|500]]

再次打包运行，成功启动，广告也已经消失

## 方法二：通过定位广告逐个击破
### 1. 固定在屏幕中上方的广告
此处附一张图片
此类广告最简单，只需修改该Activity的layout的xml配置文件即可
#### 定位Activity
```sh
dumpsys activity top | grep "com.shenyaocn.android.usbcamera" | grep "ACTIVITY" -A -0
```

输出：
`ACTIVITY com.shenyaocn.android.usbcamera/.MainActivity 2066599 pid=26465`

#### 找到并编辑布局

> 一般apk的layout信息会放在./res/layout/xxx.xml

该apk文件中找不到对应layout文件夹
通过查看**资源索引文件** `resources.arsc` 找到 `./layout/layout.xml`

```xml
<path name="activity_main">res/v9.xml</path>
```

便可以找到对应的xml文件
搜索关键字 `ad` ，找到对应节点

```xml
 <FrameLayout
     android:gravity="center"
     android:orientation="horizontal"
     android:id="@id/adcontainer"
     android:layout_width="match_parent"
     android:layout_height="wrap_content" />
```

此时有两种方法
1. 设置其高度为0 —— 最简单 `android:layout_height="0dp"` 即可
2. **直接删除** —— 根除，但是相对复杂，提供给追求完美的程序员

此时安装并启动该app，因为找不到对应节点信息，在获取该节点的时候会抛出空指针，如果该app处理了空指针异常，那皆大欢喜，如果因为异常而崩溃，此时根据[[用MT管理器和adb工具去除谷歌广告#附：通过崩溃堆栈信息找到对应字节码|堆栈信息]]找到对应smali代码，删除即可。

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231121094743796.png]]

打开 `classes.dex` - `com.shenyaocn.android.usbcamera.MainActivity`

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231121124423935.png]]

`0x7f090045` 下面会用到
删除：
`invoke-virtual {v2, v6}, Landroid/view/ViewGroup;->addView(Landroid/view/View;)V`

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/屏幕截图 2023-11-21 124035.png]]

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231121124532288.png|500]]

删除该代码（错误的，该代码并没有使用v4）
`invoke-virtual {v1, v15}, Landroid/view/View;->setVisibility(I)V`
跟着 `goto/16 :goto_37a` 往下走，并根据转换成的java文件继续分析

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231121131217140.png]]

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231121131038974.png]]

就是这了，但是观察到下面还有一个

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231121131255407.png|500]]

再继续寻找，跟着逻辑往下，就找到了
![[用MT管理器和adb工具去除谷歌广告.md_Attachments/屏幕截图 2023-11-21 132459.png]]
全部删除，完成
启动，崩溃，显然，我们需要删除与 `0x7f090045` 相关的所有代码，搜索

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231121133157050.png|400]]

已经完成一个了。

#### m0
![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231121133304096.png]]

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231121133411847.png]]

删除 23 - 31 即可

#### MainActivity
![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231121133913135.png]]
好好好，有处理空指针的操作

**OK 了**


### 2. 断开连接之后的弹窗广告
通过命令查看Activity可得：
```
ACTIVITY com.shenyaocn.android.usbcamera/com.google.android.gms.ads.AdActivity
```

去 `AndroidManifest.xml` 找到对应的Activity，删除

```xml
<activity
    android:theme="@android:style/Theme.Translucent"
    android:name="com.google.android.gms.ads.AdActivity"
    android:exported="false"/>
```

![[用MT管理器和adb工具去除谷歌广告.md_Attachments/用MT管理器和adb工具去除谷歌广告-20231121141006219.png]]

把 `com.shenyaocn.android.usbcamera.BaseAppActivity.H` 的方法体删掉即可


## 附一：应用基本信息查看

### 查看包名
用MT管理器点击文件——查看
`com.shenyaocn.android.usbcamera`
### 查看单个app的日志信息 以及 当前的Activity
进入 `adb shell`
```sh 
#获取app的pid
dumpsys activity top | grep "com.shenyaocn.android.usbcamera" | grep "ACTIVITY" -A -0
```

> PS：如果app打开就闪退，那需要手快

输出：
#### 其中包含了当前的Activity以及pid
`ACTIVITY com.shenyaocn.android.usbcamera/.MainActivity 2066599 pid=26465`

```sh
logcat -d --pid=26465
```

输出了很多内容，如果是崩溃退出了，就看最后几段内容就行

完整代码：
```sh
pid=$(dumpsys activity top | grep "com.shenyaocn.android.usbcamera" | grep "ACTIVITY" -A -0 | grep -o "pid=[0-9]*" | grep -o "[0-9]*")
logcat -d --pid=$pid
```
