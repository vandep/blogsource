---
title: NAS + DPFS-IPFS家庭云管理方案
date: 2018-07-16 15:02:03
categories:
- 技术
tags:
- p2p
- IPFS
thumbnail: http://210.73.213.194:8080/ipfs/QmZEt7mWqe8xJoiaZCYXgSRVzfcbjxFyB14WotN98pEaYx
---

### 概述
DPFS是经过改造后的IPFS私网，运行在手机端的IPFS管理功能。
NAS家庭云，拥有的大存储是作为IPFS节点的天然属性，另外其局域网内互联互通，和广域网外访问的属性，为DPFS添加和和个人主页的访问提供了良好的途径。NAS+IPFS做为文件的存储节点，通过DPFS把移动终端的文件add到NAS/IPFS节点上，并做name publish绑定到DPFS node peerId，然后即可通过ipfs gateway查看文件列表，并获取文件。
下图简介了NAS+DPFS的方案。

![NAS+DPFS简略图](\res\nas-dpfs\DPFS-NAS.png)

### 具体细节如下：

- 手机A安装DPFS demo
- NAS上部署改造后的IPFS进程
- 手机A和NAS在同一个局域网时，通过DPFS向NAS上add一个或多个文件，并做name publish
- 其他设备（手机、PC、笔记本....）无论在何时何处都可以通过个人主页（http://<gateway>/ipns/DPFSnodePeerId）查看所有在NAS/IPFS节点上的文件，也可以随时查看和下载所有文件
- 

### 示例
手机端DPFS demo上传文件到NAS

1.长按需要上传的文件，弹出menu
![DPFS-menu](\res\nas-dpfs\dpfs-menu.png)
2.点击‘Add to home page’即可执行add操作
![DPFS-add](\res\nas-dpfs\dpfs-add.png)
3.更新完成
![DPFS-publish](\res\nas-dpfs\dpfs-publish.png)

4.可以通过如下url查看添加到nas上的文件
http://210.73.213.194:8080/ipns/QmSyv5FikH4b8ihrbBXQBe6z8bZr2fUP1Ta4jMeFBHyRWf

