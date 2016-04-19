---
layout: post
title: "如何在Mac系统上搭建PHP开发环境"
date: 2015-10-22 20:28:05 +0800
comments: true
categories: Mac
---
##前言
现在有个项目后台用php做的，所以就想学习一下php，研究了差不多一天，终于可以在Mac系统上运行php环境了，现在总结一下。

##运行环境
需要的软件如下：

* MAMP
* zend studio
<!--more-->
首先说说这个MAMP,
mamp是Mac系统下php运行环境 

```
   M->Mac系统
   A->Apache 一般用Tomcat
   M->mysql数据库
   P->php
```
官方网址<https://www.mamp.info/en/mamp-pro/>  ,下载完之后,单击Preferences，选择Ports并单击Set Apache & MySQL ports to 80 & 3306建议使用的端口代替默认值。

![demo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20151022-3.png)

点击 `start servers` 开启服务，如果右上角的 `Apache Server` `Mysql server` 变成绿灯了，证明启动成功，点击中间的 `Open WebStart page`弹出网页如图:![demo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20151022-4.png)


####设置
点击perference 然后点击`webServer`,可以看到程序的根目录`mac->应用程序->MAMP->htdocs`,类似于网络托管服务器文件夹中的Public_html(默认路径可以自己修改)

默认服务器是Apache. 
![demo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20151022-5.png)


##使用Zend Studio
zend Studio是专门编写php的IDE，下载官方的 
[zend studio](http://macdown.uranusjr.com/features/) 
,
下载完后， 一路Next,在最后选择工作空间的时候，一定要选上刚才配置
MAMP的路径`mac->应用程序->MAMP->htdocs`，

然后新建一个 local php 工程![demo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20151022-6.png)

然后在index.php文件中编写如下代码:

```
	 <?php
	 echo  "Hello world";
	 echo "我成功了，我好高兴啊"
 	 ?>
```

然后在地址栏里输入 `http://localhost:8888/Demo/`,运行结果
![demo](http://7xkxhx.com1.z0.glb.clouddn.com/QQ20151022-7.png)


恭喜，你的Mac电脑也能编写php程序了!






