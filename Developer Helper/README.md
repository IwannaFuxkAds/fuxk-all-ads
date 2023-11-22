
```sh
package="cn.trinea.android.developertools"
pid=$(dumpsys activity top | grep $package | grep "ACTIVITY" -A -0 | grep -o "pid=[0-9]*" | grep -o "[0-9]*")
logcat -d --pid=$pid
```

![](DeveloperHelper.md_Attachments/DeveloperHelper-20231122131438577.png)

![](DeveloperHelper.md_Attachments/DeveloperHelper-20231122132112526.png)

这个改成 `goto :cond_32`

![](DeveloperHelper.md_Attachments/DeveloperHelper-20231122132244507.png)

直接 **return-void**

![](DeveloperHelper.md_Attachments/DeveloperHelper-20231122132517501.png)