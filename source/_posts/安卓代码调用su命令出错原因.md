---
title: 安卓代码调用su命令出错原因
---

<!-- more -->

# /system/xbin/su不存在

一般没有被root的Android手机属于这种情况。执行root即可。

# /system/xbin/su权限问题

一些Android开发板系统中含有 /system/xbin/su，但是运行权限没有给第三方应用，像下面这个权限就不对，应该加上对「其他用户」的可执行权限。

```shell
root@android:/ $ ls -l /system/xbin/su

 -rwx–x— root shell 10352 2017-07-18 21:01 su
```

修改方式：

```shell
root@android:/ $ chmod 06755 /system/xbin/su
root@android:/ $ ls -l /system/xbin/su
-rwx–x–x root shell 157400 2016-01-18 09:01 su
```

# /system/xbin/su不允许第三方应用获取root权限

```shell
root@android:/ $ su
su: uid 10061 not allowed to su
root@android:/
```

解决方案是更换不过滤uid的su文件，这样你的应用就可以正常调用了。

查看当前应用（Helloworld）的 uid(u0_a61)，并切到该用户下，然后再运行`su`就相当于该APP在调用su命令了。这样就解决了不同问题在Logcat中看到的只是如标题一样的单一的错误信息的问题。

```shell
root@android:/ # ps
u0_a61 12218 120 1259632 40368 ffffffff b6e2da80 S com.example.helloworld
root@android:/ # su u0_a61
root@android:/ $ su 
```

