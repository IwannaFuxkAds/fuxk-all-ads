从谷歌商店安装并提取安装包之后，重新安装，启动后闪退
```sh
package="com.shenyaocn.android.usbcamerapro"
pid=$(dumpsys activity top | grep $package | grep "ACTIVITY" -A -0 | grep -o "pid=[0-9]*" | grep -o "[0-9]*")
logcat -d --pid=$pid
```

![](USB_Camera_Pro.md_Attachments/USB_Camera_Pro-20231124102724427.png)

[android - Activity has leaked window that was originally added - Stack Overflow](https://stackoverflow.com/questions/2850573/activity-has-leaked-window-that-was-originally-added)

![](USB_Camera_Pro.md_Attachments/USB_Camera_Pro-20231124102808396.png)

无法验证应用程序许可证
![](USB_Camera_Pro.md_Attachments/USB_Camera_Pro-20231124102856584.png)
id: `7f1100e6`

![](USB_Camera_Pro.md_Attachments/USB_Camera_Pro-20231124102923695.png)

![](USB_Camera_Pro.md_Attachments/USB_Camera_Pro-20231124103046720.png]]![[USB_Camera_Pro.md_Attachments/USB_Camera_Pro-20231124103117512.png)

所以是在某个地方 `new b(291, obj, 0).run()`
搜索 `Lcom/shenyaocn/android/usbcamera/b` ，注意有四个参数的调用

![](USB_Camera_Pro.md_Attachments/USB_Camera_Pro-20231124103429470.png)
![](USB_Camera_Pro.md_Attachments/USB_Camera_Pro-20231124103510528.png)

是第二个方法，搜索在哪里调用了 `dontAllow` ，搜索 `->dontAllow`

![](USB_Camera_Pro.md_Attachments/USB_Camera_Pro-20231124103614471.png)

#### 第一个

![](USB_Camera_Pro.md_Attachments/USB_Camera_Pro-20231124103710937.png)

直接跳转到执行 `licenseValidator.getCallback().allow(291)` 的地方
```
...
goto :cond_20
...
:cond_20
```

成功启动
#### 第二个

![](USB_Camera_Pro.md_Attachments/USB_Camera_Pro-20231124103757713.png)

# 看一下广告的代码

![](USB_Camera_Pro.md_Attachments/USB_Camera_Pro-20231124104217173.png)

可以看到完全不会获取广告了（笑