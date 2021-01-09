---
type: "hexo"
title: hexo搭建个人博客
date: 2019-12-07 16:16:34
tags: 
	- hexo
categories: hexo
---
# 前言    


很早就有过拥有一个属于自己的网站的想法，直到一个偶然的机会在简书上看到了利用Hexo和GitHub搭建博客的教程，一个大胆的想法诞生了！(若无github账号，请注册一个[github账号](https://github.com))

#### 一、利用hexo + github搭建个人博客
##### 1.首先安装node.js
以Windows环境安装node.js为例，首先登录[node.js](https://nodejs.org/en/)官网，选择适合自己的版本进行下载，然后进行安装。(傻瓜式安装)

![image](http://upload-images.jianshu.io/upload_images/2907906-87978aebeba9430a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

##### 2.安装git(必备->后续很多步骤会用到)
登录[git官网](https://git-scm.com/)选择版本进行下载，基本一路next下去即可轻松完成安装。(傻瓜式安装)

至此，只要安装上述步骤安装，git以及node基本安装完毕，可鼠标右击测试 选择 git Bash Here打开命令行即可(用cmd的大佬请无视)！

#### 二、安装及初始化Hexo

##### 1.安装hexo
在电脑任意空白处右击选择 git bash 打开命令行
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
然后就可以使用下面的命令从npm安装Hexo：

```
npm install hexo-cli -g
```

##### 2.初始化hexo
安装完成之后，就可以选择一个自己的文件夹作为博客的根目录( 例如 C:\blog ),然后在该目录下打开命令行(git bash）

```
hexo init
```
初始化博客空间，生成博客运作所需要的文件，接下来安装依赖包

```
npm install
```

此时，你的文件夹下的目录结构应该是这样：
- scaffolds : 脚手架，用于存放我们创建文章时的模版。
- source : 用于存放我们用markdown编写的博文原文件、其他静态资源文件。
- themes : 用于存放主题，里面有我们博客的默认主题landscape。
- _config.yml : Hexo和站点的配置文件，里面可以设置博客的名字、标题、作者、链接格式等相关项。

##### 3.生成hexo博客

输入hexo generate，可缩写成hexo g

```
hexo g 
```

输入hexo s，开启服务器，访问该网址，正式体验Hexo

```
hexo s
```
启动服务器。默认情况下，访问网址为： http://localhost:4000/

问题：假如页面一直无法跳转，那么可能端口被占用了。此时我们ctrl+c停止服务器，接着输入 hexo s -p 端口号

```
hexo s -p 端口号
```

出现以下画面代表搭建成功

![image](https://images2017.cnblogs.com/blog/1108615/201710/1108615-20171021232413224-1288183746.png)

OK，至此本地搭建hexo博客成功，下面会详细讲解如何上传到github上外网访问！

#### 三、部署至github上链接外网访问

##### 1.首先在github上建立仓库 名称为 [username].github.io (请严格按照此要求建立)

---

关于什么是 SSH，请自行百度（教程不要太多..）这里直接讲一下配置步骤。 

---

请配置Deployment，在其文件夹中，找到_config.yml文件，修改repo值（在末尾）

![image](https://images2017.cnblogs.com/blog/1108615/201710/1108615-20171021235812974-84318377.png)

最后执行控制台命令

```
npm install hexo-deployer-git —save //安装部署插件
hexo d //部署到github
```

现在，我们再次访问链接：[https://username.github.io](https://lei809721586.github.io/)，就会发现这里的界面和本地的一样了。如此一来我们搭建的个人博客网站就基本完成了！

当然，还能随意更换博客主题 -> 客官请移步至[http://theme-next.iissnan.com/](http://theme-next.iissnan.com/)
