---
title: 博客之旅-穿外衣
date: 2018-06-01 10:37:44
categories:
- 技术
tags:
- blog
- hexo
thumbnail: http://210.73.213.194:8080/ipfs/QmWL6dhMvUQs1sh1J5kyMeLnU22ChVgiM2XBTvxCmsWArN


---
上一篇[博客之旅-搭架子](\2018\05\31\blog_journey-1\index.html)中已经将基本的博客搭起来了，默认是这样的


![](http://210.73.213.194:8080/ipfs/QmWL6dhMvUQs1sh1J5kyMeLnU22ChVgiM2XBTvxCmsWArN "")

我们希望它会和默认的有一些不同。这个时候就要利用theme来设置了。还记得上一篇中hexo的目录结构么

![](http://210.73.213.194:8080/ipfs/QmQqKf4Cw6Zw7q3WLnDfC9oqNdhoXJdQA2vbGehgQjLPRh)

themes目录下有本机所有的theme，默认只有landscape一种，可以在[这个地址](https://hexo.io/themes/)找到你喜欢的模板，比如这个博客挑选的[Hueman主题](https://github.com/ppoffice/hexo-theme-hueman),在[它的文档](https://github.com/ppoffice/hexo-theme-hueman/wiki)可以查看其用法。这里简单说明。

#### 安装模板
- 下载模板，直接用git clone，放到themes目录下

``` bash
$ git clone https://github.com/ppoffice/hexo-theme-hueman.git themes/hueman
```
- 修改博客的_config.yml

** 注意是blog目录下的_config.xml **
```
theme: hueman
```
- 如果需要用insight search功能的话，使用npm下载

``` bash
npm install -S hexo-generator-json-content
```

#### 修改模板
1. Menu

![](http://210.73.213.194:8080/ipfs/QmdGe3sS7nj5XqTyojyqAsRhKCUf2847PstHYpqvTqUZds)

```
    Home: /
    # Delete this row if you don't want categories in your header nav bar
    Categories:
    About: /about/index.html
```
如果不想把分类放到menu中，可以把Categories这一栏删掉
关于页面，网页链接到/about/index.html
需要在blog目录的source文件夹下建立about文件夹，然后写index.md。不知道这样的页面的md文档该如何写，可以参考了[hueman仓库](https://github.com/ppoffice/hexo-theme-hueman)的site分支的source。

也可以自定义menu，比如加入LiKe： /like/index.html,同about一样的做法。唯一需要多出的一步是，在languages的zh-cn.xml中加入Like的中文

2. customize

可以定义侧边栏的位置、主题颜色、高亮模式、logo样式、是否显示item的缩略图、social links的链接等

3. widgets

定义侧边栏项目

    - catalog
    - recent_posts
    - category
    - archive
    - tag
    - tagcloud
    - links

4. search

支持insight、swiftype、baidu三种搜索引擎，选择自己所需要的配置为true即可

5. comment

评论模块

6. share

 分享按钮，选项包括default、AddToAny、JiaThis、bdshare，具体效果可以参考
 [文档](https://github.com/ppoffice/hexo-theme-hueman/wiki/Share)



模板的配置，基本上就是这些，github上的[它的文档](https://github.com/ppoffice/hexo-theme-hueman/wiki)讲的很清楚，另外还有一些如插件之类的配置，可以自己摸索！

#### 最后的最后---性能

由于是托管在GitHub上的，带宽和访问量必然收到一定的限制，毕竟是免费的东西，方可少的话，还是够用的。另外也可以托管到放到[码云OsChina](https://gitee.com/)上，可以将github上的博客托管迁移到Oschina。

网页部署好之后发现，会有一些请求无法完成，有一些浏览器，特别是手机上，一直在进度条上。查看了下开发者选项，发现有一些js的请求，是用的http方式请求的。由于github上托管时勾选了https选项。在github上提了一个[issue](https://github.com/ppoffice/hexo-theme-hueman/issues/222)，将持续关注它
*2018/6/4:
  PPoffice已经将我提交的issue标记为bug，已经修复
  另外，count.js的的请求会有pending，由于没有配置disques评论相关的内容，所以可以现在_config.yml中关闭默认的comment配置*