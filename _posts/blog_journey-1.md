---
title: 博客之旅--搭架子
date: 2018-05-31 18:32:23
categories:
- 技术
tags:
- blog
- hexo
- IPFS
thumbnail:  http://210.73.213.194:8080/ipfs/QmTcycNzSyERKj3NQckZBZaYUgkN4mJsjuRAzjYrmgdfqQ
---
![光影梅花](\res\blog-joureny-1\光影梅花.JPG)
#### 心血来潮
最近做了一个利用去中心化存储技术[IPFS](https://ipfs.io/)实现dweb的项目，将个人内容发布到IPFS上，生成个人主页展示。
于是就有了一个用IPFS托管博客网页的想法，具体做法应该是：

```bash
$ ipfs add <index.html> #将博客静态网页add到ipfs上，得到hash
$ ipfs name publish <hash> #将博客的hash和IPFS namespace绑定
```
这样操作就可以直接通过IPNS来查看网页，链接可能是这样的：

```
http://<IPFS http Gateway>/ipns/<peerid>
```
可是这样的主页地址会是一个很长的链接，譬如：

```
http://<xxx.xx.xxx.xxx:8080>/ipns/QmSrYNxg3RdhdyyGYNBSqdH1wdRGqf1ENhXGQaHL8rACLb/
```
这样的一串链接很难映射到一个域名上。因此，退而求其次，利用github page托管静态网页，把图片/音乐/视频等大体积资源文件放在IPFS上，免受了github page空间的限制

---

#### 把搭个人博客的坑踩一遍

网络上有很多关于用[github](https://github.com) page 托管网页，用 [Hexo](https://hexo.io/)生成博客静态网页做法的文章。花了一些时间，把所有的步骤走了一遍，搭起来这个架子，期间也遇到了一些问题，简单记录：

1. 本地搭建博客生成工具Hexo
- 安装node.js 

不同的系统安装方式不太一样，windows上[直接下载](https://nodejs.org/zh-cn/)并安装，安装的时候默认把环境变量配置完成，用如下命令，验证是否把nodejs和npm（下载工具）安装成功，如果没有及时生效，可以重启一次电脑

```bash
$ node -v
$ npm -v
```
*额外的，安装完之后可以建立2个文件夹，并配置下载资源存储路径，
这样安装Hexo等module的时候，就可以方便地找到它。配置如下:*

```
$ npm config set prefix "installpath\nodejs\node_global"
$ npm config set cache "installpath\nodejs\node_cache"
```

- 安装git

```
程序员必备版本控制工具，非安装用户安装方式自行百度
```

- 安装Hexo

用npm直接下载到node_global目录下：

```bash
$ npm install -g hexo-cli
或简写 $ npm i -g hexo
```
2. 建立本地博客系统

在本地路径下建立一个文件夹用于存放博客系统的文件路径
```bash
$ cd <blog path>
$ hexo init   #初始化hexo
```
![生成的目录结构](\res\blog-joureny-1\list.PNG)

其中：
- node_modules：是依赖包
- public：存放的是生成的页面
- scaffolds：命令生成文章等的模板
- source：用命令创建的各种文章
- themes：主题
- _config.yml：整个博客的配置
- db.json：source解析所得到的
- package.json：项目所需模块项目的配置信息

在本地生成了一个默认的博客网页

```
$ hexo generate(简写hexo g) #生成静态网页
```

   开启本地server，默认可以通过localhost:4000来查看博客网页

```bash
$ hexo server 
```

3.简单修改网页配置
```
# Site
title: 可可冰的奇幻之旅
subtitle:
description:
keywords:
author: KekBin
language: zh-CN
timezone:
```
4. 将网页托管到GitHub上
- 在github上建立一个工程，如 https://github.com/vandep/vandep.github.io
- 用编辑器打开你的blog项目，修改_config.yml文件deploy配置

```
deploy:
  type: git
  repo: https://github.com/vandep/vandep.github.io.git
  branch: master

```
*此处有坑：
  type/repo/branch的冒号后面都必须有一个控制，否则发布的时候会报错
  type/repo/branch和deploy的缩进必须是一个TAB*

- 安装deploy工具
```bash
$ npm install hexo-deployer-git --save
```
- 发布网页
```
$ hexo deploy
```
5. 绑定域名（非必需）

- 如果你有一个域名

```
如果没有域名可以直接通过github.io，如https://vandep.github.io来访问
```

- 在github的blog工程下绑定域名

```
在工程中，如https://github.com/vandep/vandep.github.io
的settings下github pages一栏中cutom domain添加你的域名，save
```
- 在你的域名服务器上添加一条CNAME

![域名配置](\res\blog-joureny-1\DNS.PNG)

```
一条主机名为www， value为vandep.github.io，这样允许带www来访问自己的域名

一条主机名为@，value为vandep.github.io，允许不带www访问自己的域名

可是godaddy中不允许CNAME的主机名使用@，所以只能添加一条A记录：

ping vandep.github.io，得到一个ip，
添加A记录主机名为@， 指为IP

这样的坏处就是是固定主机的，可能ip会变，也没法CDN加速

```
- 在CNAME文件中添加域名 如 kekbin.com

*在CNAME文件中添加域名放在本机 blog 根目录下的source文件夹中
否则每次重新deploy的时候，都会重新生成CNAME上传，这样需要重新在github上添加域名。*

6.每次修改网页的内容，重新发布

一键生成并发布
```bash
$ hexo g -d
```
或者先在本地生成，启动本地server，在localhost:4000验证之后再发布
```bash
$ hexo g
#验证之后
$ hexo d
```

至此，简单的架子搭好，架子搭好之后，还需要漂亮的外衣，这就需要借助theme了，下篇介绍。










