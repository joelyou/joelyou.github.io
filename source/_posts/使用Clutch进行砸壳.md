---
title: 使用Clutch进行砸壳
date: 2018-03-27 15:51:08
tags:
---

## 使用Clutch进行砸壳
从App Store 下载的 App 都是经过签名加密的，可执行文件被加上了一层“壳”，所以逆向的第一步都是砸壳，这里记录下用Clutch来砸壳，当然你可以直接从越狱市场下载app

首先准备一个越狱手机，确保你的 Mac 和 iPhone 处在同一个 WIFI 网络下
先 ssh 到 iPhone 上，看是否遇到问题，如果遇到 Connection refused的问题，就是手机没有安装OpenSSH；
去 Cydia 安装 OpenSSH 即可，安装完后再次 ssh 到 iPhone 发现已经可以了，命令：`ssh root@ip地址` 默认密码为alpine  (当然可以自行修改的)

怎么安装Clutch？

1. 下载Clutch，将其放在电脑的某个文件，比如桌面
2. 终端进入有Clutch的目录下，执行命令`scp /Users/virgilTang/desktop/Clutch root@ip地址:/usr/bin/`,目的是将Clutch copy 到 iPhone 的 /usr/bin/ 目录下面
3. 修改 clutch 为最高权限 777,记住是在手机的/usr/bin/目录下，
   执行命令:chmod 777 ./Clutch
   查看是否修改成功
   
   `iPhone:/usr/bin root# ls -al | grep Clutch
-rwxrwxrwx 1 root   wheel 1232832 Nov  9  2016 Clutch`
可以看到已经是最高权限了。

> 如果是ssh 直接进的手机
   ![](/img/15221193099333.jpg)
则执行 两次cd..  
![](/img/15221194196979.jpg)
就能看到usr目录了，然后cd /usr/bin/目录下
![](/img/15221194911131.jpg)
> 就能看到之前复制的Clutch了；

4. 接下来是使用 clutch 了，执行命令：`Clutch -i`
   ![](/img/15221200911669.jpg)
输入你想要砸壳的应用 bundle identifier 进行砸壳
比如：` Clutch -d com.sina.weibo`
成功后你就可以看到已经破解完后的路径：
`DONE: /private/var/mobile/Documents/Dumped/com.sina.weibo-iOS8.0-(Clutch-2.0.4).ipa`
之后，copy .ipa 文件到你的电脑上，直接用命令`xhjdeMac-mini:~ xiaoou$ scp root@192.168.21.71:/private/var/mobile/Documents/Dumped/com.sina.weibo-iOS8.0-(Clutch-2.0.4).ipa ~/Desktop/`  你会发现报错：
`-bash: syntax error near unexpected token `('`
那怎么办？
先修改名字，再拷贝
![](/img/15221248109688.jpg)
这里括号前要加\转义，不然又会有问题  参考：[连接](https://my.oschina.net/iq19900204/blog/822449)

然后新建个终端拷贝到桌面上：
![](/img/15221250767903.jpg)
然后桌面就可以看到weibo.ipa，其实就和我们到越狱平台上下载的ipa是一样的已经砸壳了；


