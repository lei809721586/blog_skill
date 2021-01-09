---
title: phpStorm2019 Mac版破解(亲测有效)
date: 2019-12-13 22:54:13
tags:
	- phpStorm
categories: phpStorm
---

#### 一、 准备工作


1，从官网下载安装包：[链接](https://www.jetbrains.com/go/)

2，在网盘获取jar包：[链接](https://pan.baidu.com/s/1tEoxeN3UIEaPhoZQDW8MWQ): 密码:774c


#### 二、安装


1. 双击安装包安装PhpStorm。
2. 打开**访达**，在**应用程序**里面找到PhpStorm；右击，选择**显示包含内容**

![image](https://img.qichuanqing.cn/img/phpstrom/1.png)

3. 将网盘下载的jar包放入bin目录下

![image](https://img.qichuanqing.cn/img/phpstrom/2.png)

#### 三、破解

1. 运行PhpStorm，进行一些简单个性化的设置之后，选择试用30天（Evaluate for free）

![image](https://img.qichuanqing.cn/img/phpstrom/3.png)

2. 右下选择Configure，打开Edit Custom VM Options

![image](https://img.qichuanqing.cn/img/phpstrom/4.png)

3. 在最后一行加入-javaagent:/Applications/PhpStorm.app/Contents/bin/jetbrains-agent.jar（前面是路径，后面是文件名。），点击Save保存
4. 最后再在Configure里面打开Manage License，选择License Server，输入**http://jetbrains-license-server**网址，选择Activate进行激活




<html>
    <p style="color:red">
    ps:如果破解失败，可以尝试先创建一个项目，在top栏里面依次选择**Help->Edit Custom VM** Options，它会提醒你是否创建，创建，然后重复上面的编辑VM操作。然后重启PhpStorm，选择Register，输入网址激活即可。
    </p>
</html>

![image](https://img.qichuanqing.cn/img/phpstrom/5.png)


##### 创建一个工程之后再打开VM编辑


![image](https://img.qichuanqing.cn/img/phpstrom/6.png)


![image](https://img.qichuanqing.cn/img/phpstrom/7.png)

![image](https://img.qichuanqing.cn/img/phpstrom/8.png)

###### 输入网址激活即可！