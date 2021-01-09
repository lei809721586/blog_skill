---
title: 安装Jenkins与java
date: 2019-12-07 17:16:10
tags:
	- Jenkins
categories: Jenkins
---

```
服务器系统：CentOS 7.x
Jenkins版本： 2.121.3-1.1
Jenkins官网：https://jenkins.io/zh/ 
```


<!--在这里插入内容-->
###  1. 需要安装以下软件：

Java 8 ( JRE 或者 JDK 都可以)

Docker （导航到网站顶部的Get Docker链接以访问适合您平台的Docker下载！


### 在线安装Java 8（已有JRE 或者 JDK 略过）


```
yum search java | grep jdk
```


![image](https://dev.qichuanqing.cn/img/java1.jpg)

```
yum install java-1.8.0-openjdk.x86_64

```

安装Java1.8

按两次Y确认


```
cd /usr/lib/jvm
```

进入安装目录

![image](https://dev.qichuanqing.cn/img/java2.png)


```
vim /etc/profile
```

### 配置java环境变量


```
#set java environment

JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```


```
source /etc/profile
```

查看java版本

![image](https://dev.qichuanqing.cn/img/java3.png)

## 下载Jenkins

1.不推荐官网下载jenkins包，慢的如乌龟一般


```
官网下载地址: https://jenkins.io/zh/doc/pipeline/tour/getting-started
```

2.Linux上下载(推荐)


```
创建目录 mkdir /usr/local/jenkins
进入目录 cd  /root/
下载：wget  http://mirrors.jenkins.io/war-stable/2.138.4/jenkins.war

```

下载完毕后，打开终端进入到下载目录


```
java -jar jenkins.war --httpPort=8080
```

打开浏览器进入链接 http://localhost:8080

页面如下：


![image](https://dev.qichuanqing.cn/img/jenkins1.png)


然后复制红色的路径，在终端输入  vi  红色路径，将文件中的密码复制出来到页面的密码处


![image](https://dev.qichuanqing.cn/img/jenkins2.png)


会提示安装插件，选择安装推荐的插件，让它自动安装即可，接下来会到如下页面

![image](https://dev.qichuanqing.cn/img/jenkins3.png)


最后输入你的管理员信息，至此jebkins安装完成，请阅读下篇配置jenkins达成自动化部署！












