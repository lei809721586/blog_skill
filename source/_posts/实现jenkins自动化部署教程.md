---
title: 实现jenkins自动化部署教程
date: 2019-12-07 19:12:01
tags:
	- Jenkins
categories: Jenkins
---

### 前言


<html>
在上一篇教程中已经详细介绍过如何安装jdk与jenkins，那么本篇将详细展示如何实现jenkins的自动化部署教程

</html>



#### 1.必备
```
代码管理工具：git
代码托管平台：码云或者github
```

成功安装jenkins后请登录你输入的管理员账号密码到如下界面：

![image](http://img.mymealwell.cn/demo/jenkins/117.png)

1.安装 	Publish Over SSH，git ， ssh ，gitee 插件


 
```
首页左侧栏-> Manage Jenkins -> Manage Plugins 
```

![image](http://img.mymealwell.cn/demo/jenkins/23334.png)


#### 2.输入你的自定义自动化部署名称，选择第一个选项 -> 构建一个自由风格的软件项目 , 点击确定

2.1 源码管理，这里因为还没有配置公钥所以报错

![image](http://img.mymealwell.cn/demo/jenkins/203924.png)


ps:这里需要添加一个ssh公钥

建议去您的linux服务器上设置一个 jenkins账号
如：

```
useradd jenkins
passwd "您的密码"
再次确认输入密码

为新用户添加sudo权限：
vim /etc/sudoers
保存 mq!
```
![image](http://img.mymealwell.cn/demo/jenkins/33333333335006.png)


```
用jenkins用户登录服务器

su jenkins
输入您的密码

生成ssh公钥
ssh-keygen -t rsa
# 三次回车即可生成 ssh key

cat ~/.ssh/id_rsa.pub
复制您的公钥到码云上
```
![image](http://img.mymealwell.cn/demo/jenkins/3333205455.png)

![image](http://img.mymealwell.cn/demo/jenkins/888888888881229205304.png)

回到上个步骤，新增全局凭证
![image](http://img.mymealwell.cn/demo/jenkins/112333305806.png)

这样就可以了
![image](http://img.mymealwell.cn/demo/jenkins/QQwancheng5849.png)

接下来选择  “构建触发器”，选择Gitee webhook 触发构建 复制您的url到码云

![image](http://img.mymealwell.cn/demo/jenkins/goujian20.png)

在码云上的项目 -》 管理 -》 添加 webhook 

![image](http://img.mymealwell.cn/demo/jenkins/may227.png)


==这里webhook密码需要在jenkins上生成== 
生成的密码复制到码云 webhook密码出，点击添加保存即可

![image](http://img.mymealwell.cn/demo/jenkins/fffff29210509.png)


选择构建环境 =》 Execute shell
![image](http://img.mymealwell.cn/demo/jenkins/99999999999210659.png)

测试一下，先拉取一份项目到本地
![image](http://img.mymealwell.cn/demo/jenkins/a-224.png)

![image](http://img.mymealwell.cn/demo/jenkins/a-1.png)

![image](http://img.mymealwell.cn/demo/jenkins/a-33.png)

好的，本地推送push master成功，现在来看服务器上是否自动同步了呢？

在jenkins中的控制台输出可以看到是构建成功了

![image](http://img.mymealwell.cn/demo/jenkins/a-5.png)

看看服务器上是否自动同步了呢
![image](http://img.mymealwell.cn/demo/jenkins/a-6.png)

很完美，自动构建成功，本地新创建的a.txt也有了，内容也有！！！

并且码云上 webhook也有记录

![image](http://img.mymealwell.cn/demo/jenkins/a-11.png)


end




























