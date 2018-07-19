---
title: IPFS实践之初体验

date: 2018-06-06 11:05:24

thumbnail: http://210.73.213.194:8080/ipfs/QmeLRbhaxb7gbfX6RDwfQhUCrjZRfeDwfY6P3Q4WC4CoDn
categories:
- 技术

tags: 
- IPFS
---
![](\res\ipfs-1\雨后.jpg)
#### 概述
IPFS的全称是InterPlanetary File System（星际文件系统），从名称上看，这是一个很炫酷、很有野心的项目。简单地说它就是一个点对点的分布式文件系统。[官网](https://ipfs.io)和[github](https://github.com/ipfs)都可以找到所有的相关资料。建议从它的[白皮书](https://github.com/ipfs/ipfs/blob/master/papers/ipfs-cap2pfs/ipfs-p2p-file-system.pdf?raw=true)，和[直译中文版本](http://210.73.213.194:8080/ipfs/QmVTb5CkppwVdRQZnevNYTnQV95tmZAMoZFasDLtxqZn1i)开始了解，后面我们会慢慢地认识它。白皮书上指出了多个应用场景：
> 1. As a mounted global filesystem, under /ipfs and /ipns.
> 2. As a mounted personal sync folder that automatically
versions, publishes, and backs up any writes.
> 3. As an encrypted file or data sharing system.
> 4. As a versioned package manager for all software.
> 5. As the root filesystem of a Virtual Machine.
> 6. As the boot filesystem of a VM (under a hypervisor).
> 7. As a database: applications can write directly to the
Merkle DAG data model and get all the versioning,
caching, and distribution IPFS provides.
> 8. As a linked (and encrypted) communications platform.
> 9. As an integrity checked CDN for large files (without
SSL).
> 10. As an encrypted CDN.
> 11. On webpages, as a web CDN.
> 12. As a new Permanent Web where links do not die.


#### 体验
我想从第一个应用场景开始，开启我们的应用旅程。

这个场景可以把它想象成一个特别的"云盘"，这个"云盘"不依赖任何云存储商，不依赖任何平台账号，不用担心云盘提供商的内容审核，不用担心个人隐私的泄密，不用担心内容丢失或被篡改。这些特性希望通过后面的介绍，大家能够自己体会到。可以先试想下，平常我们使用云盘的几个基本需求：
- 上传文件
- 下载文件
- 分享文件

下面我们看IPFS如何作为一个“云盘”，来满足这些基本需求的。参考官方的[get-start](https://ipfs.io/docs/getting-started/).

#### 下载Demo
  
首先下载官方提供了IPFS的Demo，比较完整的版本是[Go实现](https://github.com/ipfs/go-ipfs)的，目前最新版本是[0.4.15](https://dist.ipfs.io/#go-ipfs),包含多个平台的版本，以windows 64bit系统为例，下载go-ipfs_v0.4.15_windows-amd64\go-ipfs,由于网络限制，可能有些同学无法下载，我这边上传了一个到IPFS网络上，有需要的同学可以[直接下载](http://210.73.213.194:8080/ipfs/QmULF3d3sHNecj5JDv9joaapGJ8beUgq73DaPRqrz9Pofm)。
    
这个demo中提供了Node/Cli/Http api/Http Gateway/Library/webUi功能，本文中暂使用CLI的来体验IPFS的基本功能，为了方便在cmd直接执行IPFS命令，可以将Go-ipfs加入环境变量path中。

#### 初始化

``` bash
> ipfs init
```
执行初始化之后，user路径下会多了一个.ipfs的文件夹，其中有一个config文件，记录的是IPFS的配置，配置内容后续的讲述中会慢慢涉及，此刻我们先了解的是IPFS节点的身份标识Identity，其中peerid，是本机IPFS的地址，下文中IPNS的使用中会涉及到peerid。当然，peerid可以通过命令行直接查看:

```bash
> ipfs id
```

#### 启动Deamon
执行
```bash
> ipfs daemon
```
开启deamon之后，会启动demo中包含的IPFS服务，至此，本机就可以作为一个"个人云盘"来使用了。

#### 上传文件
```bash
> ipfs add <filePath> //添加文件
> ipfs add -r <dirPath> //添加文件
```
文件添加时，大文件会按照256KB的大小去分块，分块内容存储在.ipfs repo下的blocks文件夹下，每一个块都对应一个hash。文件添加完成之后，会返回一个文件hash，文件夹是递归添加的，会将每一个文件的hash返回，最后一个返回文件夹hash。由于IPFS中，文件都是基于内容寻址的，用户不需要关心这些文件放在哪，这些hash就是用来查找和得到文件的索引。
    
#### 下载文件
想要拿到一个IPFS上的文件，只需要知道文件的hash即可。
下载命令：
```bash
> ipfs get <hash> -o <output_path>
```
或者查看命令：
```bash
> ipfs cat <hash>
```
IPFS 的所有文件都是在本地的，Pin add可以将远端节点的文件长期保留在本地，不被垃圾回收。通过 add 添加的文件默认就是pin过的。

pin还有一个作用，pin add之后，这个节点就可以同add这个文件的节点一样，作为整个文件的服务节点。其他节点在get或pin add的时候理论上可以从这2个节点的任何一个节点拿去这个文件的块。当有很多个节点都pin add这个文件时，就会增加这个文件的获取速度，也降低了文件被删除的可能。

```bash
> ipfs pin add <hash>
> ipfs pin ls //查看哪些文件在本地是持久化的。
```
#### 分享文件

现实中，用户习惯获取内容的方式都通过http的方式，让用户去敲get命令去或取文件恐怕只有极客们会尝试吧。回想一下，云盘分享文件的做法，生成一个http url，把链接分享给需要的人。

IPFS demo也提供了一个http gateway，官方（公网）用的是 http://ipfs.io  ,所以分享的链接可以是:

    http://ipfs.io/ipfs/<file hash>

由于某些已知原因，ipfs.io这个网关在国内是没法正常使用的，因此有条件的可以在自己的服务器上部署一个IPFS gateway（Ipfs 私网），这样，分享的链接可以是

	http://<gateway_host>/ipfs/<file hash>


#### IPNS

到现在，我们分享文件还是不断地发送一个个hash组成的url，每分享一个文件就会重新生成一个url，那么可以有一固定的"网址”给我们来看你的分享呢？当然可以。这就需要借助IPNS。具体做法是
```bash
> ipfs name publish <file_hash>
```
这条命令相当于把文件的hash和IPFS的节点绑定，这样就可以通过一个不变的地址来访问publish的文件 

	http://<gateway>/ipns/<peerid>

peerid即之前介绍的本节点的唯一标识。
这样做的有一个很大的缺陷，就是每次这个链接指向的都是最后一条publish的内容。但是有解决方法，大家可以查看本博客的[关于页](../../../../about/index.html)，我的[IPFS主页](http://210.73.213.194:8080/ipns/Qmd4KAjNMgbeuN86iQK2MSvEHjGgSc7FrcHpkhro8Qqty7/)。实现方式其实很简单，就是将将分享文件url的列表publish到ipns上。后续可能会再写一篇博文介绍做法。

