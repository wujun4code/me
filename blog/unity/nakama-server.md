---
date: "2018-09-02"
title: "Nakama 尝鲜 - 游戏服务端的一些调研"
category: "Unity"
---

<!-- TOC -->

- [手游发展史简谈](#手游发展史简谈)
- [游戏服务端选型与对比](#游戏服务端选型与对比)
- [闭源 SaaS 服务](#闭源-saas-服务)
    - [Photon Server](#photon-server)
    - [SmartFoxServer](#smartfoxserver)
- [开源服务端](#开源服务端)
    - [ComBlockEngine(原 kbengine)](#comblockengine原-kbengine)
    - [[Pomelo](https://github.com/NetEase/pomelo) - 网易的玩具](#pomelohttpsgithubcomneteasepomelo---网易的玩具)
- [[Nakama](https://github.com/heroiclabs/nakama) - 本文的主角终于登场了](#nakamahttpsgithubcomheroiclabsnakama---本文的主角终于登场了)
- [尝试从零开始部署 Nakama](#尝试从零开始部署-nakama)
    - [必备知识](#必备知识)
- [总结](#总结)

<!-- /TOC -->


## 手游发展史简谈

自从 iOS 的崛起，加上《愤怒的小鸟》的推波助澜，手游成为了自 2010 年之后最大的战场，抛开 3A 大作不说（大多数都是弱联网的烧显卡的单机游戏），也暂时不谈主机游戏的固有阵地，手游的热门类型变化实在是太快了，而伴随而来的就是技术上的基础设施和基础架构的演变。

上古时期(约公元 2007 - 2011)，手游几乎都是在玩法上创新的单机游戏，并且苹果还饶有兴致的推出了 Game Center 的 API 框架来帮助厂商快速的建立自己的好友关系网，尽快的帮助游戏推广，加上《植物大战僵尸》，《愤怒的小鸟》，以及迪斯尼出品的《Swampy》，以及《无尽之刃》几乎从手游的触屏特性和画风，画面上占据了绝对的口碑和市场，这个时代和 PC 游戏的发展历程简直一模一样，但是 PC 网游的时代直到 2003 年魔兽世界的横扫千军才正式开启，而手机网游则来的更迅速，在 8012 年的 App Store 单机游戏排行榜上除了 《纪念碑谷》还可能占据前 20 的位置，其余的单机游戏霸榜能力实在是不行，从网易的《梦幻西游手游》开始，手机网游正式接管单机手游，接下来进入正题：

## 游戏服务端选型与对比

## 闭源 SaaS 服务

这里用 SaaS 可能不要确切，但是仔细想想，他们也是提供了一些软件层面的接口给你调用，你完全不用编写任何服务端的逻辑，不用进行服务器运维，就可以开发出一款游戏，这约等于是 SaaS 了，

### Photon Server

比较有代表性是 Photon Server，这个几乎是与 Unity3D 同生共死的云服务，它最大的卖点就是 Unity3D 的插件和客户端 Demo 做的比较完善，易用性尚可，但是缺点也十分巨大：如果你想自定义服务端，首先，他就只扔给你一个 exe 程序，你需要自己租用云服务器（并且只能是 Windows Server），然后在你的服务器上双击启动这个 exe ，然后你自己做集群，你自己做内网互通，你自己做运维，也仅仅是因为它客户端做的比较有噱头，好多小厂商（大概也只有国外小游戏厂商会）热衷于它，我个人觉得，相对于国内复杂的游戏热度和游戏风格，这个服务在国内大部分游戏公司都是自建服务端的情况下，前景十分不看好。


### SmartFoxServer

这个就更老了，它现在还支持 Flash 客户端，简直是业界一股清流（(⊙o⊙)…），它是由 Java 构建的，性能自然不是问题，问题是冗余的东西太多，并且价格不便宜，Java 的还闭源，这就令人难过了。

不过相对来说，Java 程序员好找，这个还是比较推荐给大多数的中小厂商使用，如果你扛得住它的授权费用的话。

## 开源服务端 

### ComBlockEngine(原 kbengine)

第一个需要谈的就是大名鼎鼎的是 [kbengine](https://github.com/kbengine/kbengine)，它是来自于杭州一个团队开源的一款经久不衰的 MMORPG 游戏服务端，已经被众多大型的游戏厂商检验过的框架，假设你的团队当中有人写对 C++ 比较擅长的话，别犹豫了，用这个引擎绝对是首选，另外它允许你用 Python 构建你自己的服务端逻辑，并且还支持服务端的热更新，这个就很性感了，对于瞬息万变的手游市场，热更新绝不是刚需，是必需。

他几乎是现在开源服务端引擎里面最佳的选择，并且现在开始商业化了，目前来看他的商业化的内容还没有做到托管这一层，可能是看到了现在云服务器已经是末日行业，马太效应严重，他们也就不愿意做了，费力不讨好，而转为做技术支持，培训教育，驻场开发等诸多服务上的内容，个人觉得坚持做的话，收入还是会不错的。

但是有一说一，有一些隐患就是 C++ 程序员不好招，另外它的运维压力很大，不是高手可能处理不了，大概搜了一下，docker 也没有官方给予支持，不过他们正在研发商业版，商业版应该会面向容器技术友好，否则运维压力实在不小。

> 游戏运维是什么？不存在的。一台机器一个服务器，一台机器一个地图，一台机器一个服务，挂了？重启！

### [Pomelo](https://github.com/NetEase/pomelo) - 网易的玩具
这个是网易用 NodeJS 编写的一个开源框架，可惜因为几乎不挣钱，网易宁可养猪也不维护了，但是还是有一些民间组织基于它做了一些魔改，具体的不做过多介绍，反正没人会用了，死掉的先驱者。

## [Nakama](https://github.com/heroiclabs/nakama) - 本文的主角终于登场了

这是我个人比较喜欢的一款开源产品，它亮点十足：

- Go 语言编写，分布式天然而成，性能绝对不差
- 官方支持的 Docker Compose 服务，部署真的不要太爽
- 客户端支持的足够多，Unity/Unreal/Cocos家族都支持
- 并且它官网给出了各种与 Photon Server 等诸多竞品的直接对比图，各种看上去的吊打，不给面子啊。


## 尝试从零开始部署 Nakama 

### 必备知识

- Linux 机器常规命令行操作
- [Docker](https://docs.docker.com/install/) 和 [Docker Compose](https://docs.docker.com/compose/overview/) 的使用基本知识

首先你得有一台 Linux 的云服务器（其实并不是非得 Linux，假设你司有钱，是用 Windows Server 的，那也行，都是大佬...），关于这个你可以从腾讯云、阿里云、UCloud、青云各种搞一台都行，我个人比较习惯了微软的 Azure ，创建云服务器用户体验稍微好一点。此处有一个关键点是你的云服务器一定要打开外网的 `7350` 端口(TCP)。

如下图：

![image](https://user-images.githubusercontent.com/5119542/44952731-834f5f80-aeb9-11e8-825d-13f7d103e939.png)


然后你需要 ssh 登录进去:

![image](https://user-images.githubusercontent.com/5119542/44952765-38821780-aeba-11e8-8810-abc226a7ae48.png)

然后你需要打开 [Docker Compose](https://docs.docker.com/compose/install/#prerequisites) 的官方网站，安装一下 Docker Compose。此处有个坑， Ubuntu 的 apt-get 安装的包是古老的版本，千万不要安装，请用 [Docker Compose](https://docs.docker.com/compose/install/#prerequisites) 官网的 curl 命令安装。

安装完毕之后，并且确保一定成功执行了:

```sh
wujun@TMServer:~$ docker-compose --version
docker-compose version 1.22.0, build f46880fe
```

之后再继续下一步，用 `vi` 命令创建一个 `docker-compose.yml` 的文件，输入如下内容：

```
version: '3'
services:
  cockroachdb:
    container_name: cockroachdb
    image: cockroachdb/cockroach:v2.0.3
    command: start --insecure --store=attrs=ssd,path=/var/lib/cockroach/
    restart: always
    volumes:
      - data:/var/lib/cockroach
    expose:
      - "8080"
      - "26257"
    ports:
      - "26257:26257"
      - "8080:8080"
  nakama:
    container_name: nakama
    image: heroiclabs/nakama:2.0.2
    entrypoint:
      - "/bin/sh"
      - "-ecx"
      - >
          /nakama/nakama migrate up --database.address root@cockroachdb:26257 &&
          /nakama/nakama --name nakama1 --database.address root@cockroachdb:26257
    restart: always
    links:
      - "cockroachdb:db"
    depends_on:
      - cockroachdb
    volumes:
      - ./:/nakama/data
    expose:
      - "7350"
      - "7351"
    ports:
      - "7350:7350"
      - "7351:7351"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:7350/"]
      interval: 10s
      timeout: 5s
      retries: 5
volumes:
  data:
```

保存并退出编辑模式，然后在命令行里面输入:

```sh
docker-compose -f docker-compose.yml up
```

恭喜你！如果一切顺利，你将拥有一台 Nakama 的游戏服务器了，它什么都准备好了。

接下来我们下载 Nakama 的 Unity SDK，你打开 Unity IDE 在 Assets Store 里面搜索 Nakama 关键字就可以直接下载并导入：

![image](https://user-images.githubusercontent.com/5119542/44952837-286b3780-aebc-11e8-8175-d47d25f53b64.png)

直接编写一个脚本：

```cs
using System.Collections;
using System.Collections.Generic;
using System.Threading.Tasks;
using Nakama;
using UnityEngine;

public class LogIn : MonoBehaviour
{
    Client client;
    // Use this for initialization
    void Start()
    {
        client = new Client("defaultkey", "这里填写你云服务器的外网 ip 地址", 7350, false);
    }

    public void AuthenticateAsync()
    {
        const string email = "hello@example.com";
        const string password = "somesupersecretpassword";
        client.AuthenticateEmailAsync(email, password).ContinueWith(t =>
        {
            var session = t.Result;
            Debug.LogFormat("Authenticated session: {0}", session);
        });

    }

    // Update is called once per frame
    void Update()
    {

    }
}

```

绑定一个 Button 去调用 `AuthenticateAsync`:

就可以在运行时看到如下内容：

![image](https://user-images.githubusercontent.com/5119542/44952860-741de100-aebc-11e8-8505-d6867ea315b4.png)


如果打印出来了日志，那么说明一切就绪了你可以开发自己的游戏了，如果你需要自定义服务端，那你最好具备如下知识：

- 会写 Lua，基本上游戏程序员都需要学会这个
- 阅读官方文档，会告诉你如何根据提供的各种 Hook 函数来自定义逻辑


## 总结
在手机进入网游时代，大厂（腾讯，网易，完美等）肯定是自己研发服务端，甚至网易腾讯都会有自己研发的客户端引擎，而这些闭源服务和开源产品肯定不会入他们的法眼，而小厂商在决定使用哪些的时候，一定要衡量一下，开箱即用的服务和需要一定的运维能力的开元产品之间做出符合自己优势的选择，还有一个就是价格，闭源服务的价格在我看来都好黑，再次安利一下 Nakama 这个引擎，好感度依然如初见。




