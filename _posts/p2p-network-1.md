---
title: P2P网络由浅入深—基于局域网模型
date: 2018-06-26 16:43:53
tags:
- p2p
- IPFS
- libp2p
categories:
- 技术
thumbnail: http://210.73.213.194:8080/ipfs/QmegcHbDSKevGUWeaPfV4scsKhcaURw2PhSTXbAcSbLy15
---

最近开始接触IPFS，前面博文中也略有提到过，它是一个去中心化的分布式网络文件系统。其核心之一就是P2P网络（peer-to-peer network），可以说P2P网络是IPFS的基础。IPFS项目组为P2P网络构建了一个单独的开源项目[libp2p](https://github.com/libp2p/libp2p),使其不仅可以用在IPFS项目中，也可以用于构建其他的P2P网络。


## p2p网络与中心化网络

本文不想对两者的优劣进行比较，因为目前似乎比较它们的优劣也没有任何意义，毕竟现实情景下，两者是无法互相替代的。至少目前热点追捧的去中心化，还无法替代中心化的功能。

计算机网络的初衷就是建立一个相互通信的对等的计算机网络，但是由于历史问题连接在网络的计算机配置不高，计算能力和存储能力不足，因此相当长一段时间我们更倾向于使用基于客户端（client）和服务器（server）的C/S模式及浏览器（Browser）和服务器（server）的B/S模式:

![中心化网络简单示意](\res\p2p-network-1\中心化网络示意.png)

在libp2p的白皮书中，列举了如下选用cs建构的理由：
> - The bandwidth inside a data center is considerably higher than that available for clients to connect to each other.
> - Data center resources are considerably cheaper, due to efficient usage and bulk stocking.
> - It makes it easier for the developer and system admin to have fine grained control over the application.
> - It reduces the number of heterogeneous systems to be handled (although the number is still considerable).
> - Systems like NAT make it really hard for client machines to find and talk with each other, forcing a developer to perform very clever hacks to traverse these obstacles.
> - Protocols started to be designed with the assumption that a developer will create a client-server application from the start.

简单地概括就是高带宽、低成本、易维护、穿透性强、易开发。
其实这些理由也只是相对而言，例如低成本，服务器打带宽和电费支出，远高于对等节点的支出。所谓易维护，如果所有服务器宕机的话，这个服务是暂停的。

![P2P简单示意](\res\p2p-network-1\p2p简单拓扑.png)

当然上图只是简单的示意，具体P2P的网络拓扑远比图示更多样、更复杂。P2P网络中的所有节点都是对等的，各台计算机有相同的功能，无主从之分，一台计算机既可作为服务器，设定共享资源供网络中其他计算机所使用，又可以作为工作站，整个网络一般来说不依赖专用的集中服务器，也没有专用的工作站。网络中的每一台计算机既能充当网络服务的请求者，又对其它计算机的请求做出响应，提供资源、服务和内容。

听起来，比中心化的网络更加诱人对吧。这样的对等网络需要解决的事情还有很多，如NAT后面的节点彼此发现和可达、各个节点的吞吐量是否能达到人们对性能的可忍受度。
***
## 基于局域网环境的P2P模型

之前一直在做局域网的文件传输工具的开发，其实局域网文件传输所需建立的网络，从广泛意义上讲，也是一个P2P的网络，只不过它的情况会比广域网上的P2P要简单的多。

![局域网下的p2p拓扑](\res\p2p-network-1\局域网下的p2p.png)
- 无需考虑NAT穿透，因为每个节点都是在同一个网关下
- 节点彼此发现简单
- 拓扑简单

但是简单的网络结构也需要实现P2P网络的几个关键问题

##### 节点路由
本文的前提是在同一个局域网下，所以路由的问题就比较简单了，所有的数据传输经路由器转发。

*此处不必把路由器和中心化服务器进行混淆，路由器在这里只充当网络数据的桥梁，对网络中所有的数据进行透传。*

##### 节点之间的发现
网络中的节点，起初就像是不认识的陌生人，互相是不知道对方的任何信息的。所以，如果要知道网络中某个节点的具体信息，甚至是传输东西给对方，就需要有一个沟通的途径，让彼此能够发现自己。以往我们做的众多的局域网传输的经验中，有以下几种方式，能够发现设备：
1. 热点发现

    类似于WIFI的发现机制，通过无线电信号，发现方启动热点扫描，发现热点设备。发现之后，这样热点的启动设备和发现设备就组成了一个1对1的网络。但是如果多个设备都连入这个热点，启动热点的设备，就变成一个路由，所有其它节点之间的消息都需要热点设备进行转发，因此它已经丧失了P2P的意义。
2. 蓝牙发现
    
    借助蓝牙的发现机制，发现对方，然后通过蓝牙通信告知网络基本信息及地址。
3. UDP广播

    网络中的节点通过某个端口不断地向全网广播自己的信息，网络中的所有节点都监听这个端口，这样收到广播的节点可以进行回复，这样双方就知道对方的基本信息和地址。为了网络信息的安全性，可以对信息进行加密，这样非恶意节点才可以解密该信息。额外的风险在于，恶意节点也可以不断地向该端口发送广播，污染网络，使消息阻塞，无法扫描发现。如消息格式：
    
```    
    magic(4)|  sequence(8) | version(1) | type(1) | enc(body)
```

##### 节点连接
发现节点之后，彼此了解了各自的地址（如IP），在局域网中，就可以通过ip和固定端口直接建立TCP连接。

##### 节点数据交互

连接之后可以建立信令通道，传递控制信息。控制通道上跑的信息可以是自定义的任何这个网络中所有节点认可的协议，传递二进制、JSON等数据皆可。

另外在文件共享的网络中，还必须有一个数据通道，用于交互数据信息。

数据通道中用于传输文件的协议会有很多中选择，如HTTP(HTTPS)、FTP、MTP等
。

在这个简单的P2P网络中，每一个节点即是server端（如Http server），用于发送文件，也可以是client端（如http client），用于请求文件内容。所有节点都是对等的。文件服务无需借助于一个云服务单元来上传下载。

以上的所讲局域网络模型，对真实的P2P网络其实有很大的不同，但是具有理解上的借鉴意义。当中可能也有些不够准确的地方，随着学习的了解的深入，慢慢剖析吧。