---
date: "2018-09-04"
title: "RocketChat - Slack 最好的替代品"
category: "Tools"
---

<!-- TOC -->

- [工作的线上沟通用什么？](#工作的线上沟通用什么)
    - [国内三巨头](#国内三巨头)
        - [QQ](#qq)
        - [微信](#微信)
        - [钉钉](#钉钉)
    - [国外 Slack 独秀](#国外-slack-独秀)
- [[RocketChat](https://rocket.chat/) 的试用](#rocketchathttpsrocketchat-的试用)
- [效果图](#效果图)
- [总结](#总结)

<!-- /TOC -->

## 工作的线上沟通用什么？

我们先来列举一下国内外比较流行的或者说知名的在线办公聊天软件和 SaaS 服务：

### 国内三巨头

- QQ 
- 微信
- 钉钉

#### QQ 

先别看它最老，但是目前很多企业还是用 QQ，这个很多可能多的超乎想象。我有一个前同事创业之后就直接用 QQ ，每周在 QQ 公告栏里面修改本周五之前要完成的计划，谁完成了自己上去修改公告板，就是这么简单暴力，还挺有效果的。

#### 微信
本身根本不具备办公技能，但是微信的产品团队从 2016 年开始很在乎 B 端用户的市场，但是一直不温不火，实际上，打开微信的企业版的后台管理界面：

![image](https://user-images.githubusercontent.com/5119542/45018790-5fc51a00-b05d-11e8-8aed-b5b64b93631a.png)

不难看出，微信的合作伙伴很多，只是在闷声发大财，现在企业微信也从微信中隔离出来，我个人觉得它的出路至少比阿里的钉钉要更好。

#### 钉钉

钉钉是传统企业老板最喜欢的，假设他们真正体验过，就会感受到一股做皇帝的感觉，每天的日报，周报，签到功能，已读回执，谁未读消息一清二楚，简直不要太爽。

但是它最大的黑点也就是真正的用户不喜欢它对人的束缚，它服务于老板，而不是服务于它真正的用户，这个我觉得局限了它的发展，不好说他一定是这个市场最大的赢家，至少在体验上，我没见过一个用过钉钉还会继续在新的公司推广钉钉的，因为真的是恨之入骨。


### 国外 Slack 独秀

说 Slack 之前，我们先说说微软的 MSN 和 Skype，这俩货死因是一样的，根本不知道它到底要做什么，社交得导入 Facebook 关系人，Skype 唯一最好的地方就是多平台多设备的多人会议，但是它本身的 UI 和协同方向的功能太少，另外 Web 版直到 2015 年才勉强 Beta，黄花菜都入土了。

说到 Slack，可能一部分在国内创新型的互联网企业都至少是了解，哪怕试用，更有很多知名创业企业是重度使用的比如某乎，它的优点很明显

- UI 现代美化
- 多平台支撑优秀
- 稳定好用
- 频道/上下文很好管理和搜索
- 贵，对这是他的优点（手动斜眼）

有一个致命的问题，它……它……它没有国内的运营权，属于被墙的状态。

接下来今天的主角要登场了：[RocketChat](https://rocket.chat/) 

它的官网就赫然写着：

> Rocket.Chat is free, unlimited and open source. Replace email, HipChat & Slack with the ultimate team chat software solution.

简单的翻译就是 ：

> RocketChat 就是 Slack 最棒的开源替代品（脸呢？不要了。）

## [RocketChat](https://rocket.chat/) 的试用

试用之前你需要掌握的技能和物理资源

- 一台 Linux 云主机
- [Docker Compose](https://docs.docker.com/compose/)的使用基础入门
- 云主机的端口映射和配置
- Vi 的基本使用技能（新建文件/编写文件/修改内容） 

直接试用，因为功能上没有特别之处，特别的功能一般的工作中根本用不上。

首先你需要一台云主机（腾讯云，阿里云满地都是）

然后还需要安装好 [Docker Compose](https://docs.docker.com/compose/)，

然后在云主机任意一个目录下，比如 [RocketChat](https://rocket.chat/) 推荐你在：

```sh
sudo mkdir /var/www/rocket.chat/
```
下新建 `docker compose.yml` 文件内容如下：

```
version: '2'

services:
  rocketchat:
    image: rocketchat/rocket.chat:latest
    restart: unless-stopped
    volumes:
      - ./uploads:/app/uploads
    environment:
      - PORT=3000
      - ROOT_URL=http://localhost:3000
      - MONGO_URL=mongodb://mongo:27017/rocketchat
      - MONGO_OPLOG_URL=mongodb://mongo:27017/local
      - MAIL_URL=smtp://smtp.email
#       - HTTP_PROXY=http://proxy.domain.com
#       - HTTPS_PROXY=http://proxy.domain.com
    depends_on:
      - mongo
    ports:
      - 3000:3000
    labels:
      - "traefik.backend=rocketchat"
      - "traefik.frontend.rule=Host: your.domain.tld"

  mongo:
    image: mongo:3.2
    restart: unless-stopped
    volumes:
     - ./data/db:/data/db
     #- ./data/dump:/dump
    command: mongod --smallfiles --oplogSize 128 --replSet rs0 --storageEngine=mmapv1
    labels:
      - "traefik.enable=false"

  # this container's job is just run the command to initialize the replica set.
  # it will run the command and remove himself (it will not stay running)
  mongo-init-replica:
    image: mongo:3.2
    command: 'mongo mongo/rocketchat --eval "rs.initiate({ _id: ''rs0'', members: [ { _id: 0, host: ''localhost:27017'' } ]})"'
    depends_on:
      - mongo

  # hubot, the popular chatbot (add the bot user first and change the password before starting this image)
  hubot:
    image: rocketchat/hubot-rocketchat:latest
    restart: unless-stopped
    environment:
      - ROCKETCHAT_URL=rocketchat:3000
      - ROCKETCHAT_ROOM=GENERAL
      - ROCKETCHAT_USER=bot
      - ROCKETCHAT_PASSWORD=botpassword
      - BOT_NAME=bot
  # you can add more scripts as you'd like here, they need to be installable by npm
      - EXTERNAL_SCRIPTS=hubot-help,hubot-seen,hubot-links,hubot-diagnostics
    depends_on:
      - rocketchat
    labels:
      - "traefik.enable=false"
    volumes:
      - ./scripts:/home/hubot/scripts
  # this is used to expose the hubot port for notifications on the host on port 3001, e.g. for hubot-jenkins-notifier
    ports:
      - 3001:8080

  #traefik:
  #  image: traefik:latest
  #  restart: unless-stopped
  #  command: traefik --docker --acme=true --acme.domains='your.domain.tld' --acme.email='your@email.tld' --acme.entrypoint=https --acme.storagefile=acme.json --defaultentrypoints=http --defaultentrypoints=https --entryPoints='Name:http Address::80 Redirect.EntryPoint:https' --entryPoints='Name:https Address::443 TLS.Certificates:'
  #  ports:
  #    - 80:80
  #    - 443:443
  #  volumes:
  #    - /var/run/docker.sock:/var/run/docker.sock
```

**此处有坑**：

请把文件内容中的 `ROOT_URL=http://localhost:3000` 一定要修改成你云主机的 IP（假设你主机有 ICP 备案以及域名的话，绑定域名也可以）

否则在真正运行的时候：图片和文件消息基本发布出去，因为这个 `ROOT_URL` 就是你文件资源访问的 Url。

然后直接启动 MongoDB 服务：

```sh
docker-compose up -d mongo
```

然偶再启动初始化服务:

```sh
docker-compose up -d mongo-init-replica
```

最后启动核心服务:

```sh
docker-compose up -d rocketchat
```

另外如果习惯了使用 Slack 各种可拓展的第三方小机器人的话，在 [RocketChat](https://rocket.chat/) 里面也有对应的解决方案:

```sh
docker-compose up -d hubot
```

更多细节可以直接看官网文档： [https://rocket.chat/docs/installation/docker-containers/docker-compose/](https://rocket.chat/docs/installation/docker-containers/docker-compose/)


假设一切部署成功之后，你可以从浏览器访问你的 [RocketChat](https://rocket.chat/)：

作为第一个访问者，系统会指导你创建第一个超级管理员，请一定确保账号和密码的安全，万一忘了密码恐怕你得自己手动连服务器上的数据库去改了（开玩笑的，其实它有其他方式），

一切必要资料填写完毕之后，你会看见一个几乎是跟 Slack 一模一样的聊天办公界面出现在你的浏览器，恭喜你，你拥有了一份自己的 Slack（啥？你问我怎么做集群？我这里信号不好……）。


## 效果图

![image](https://user-images.githubusercontent.com/5119542/45021026-15469c00-b063-11e8-8ea3-8c917818d984.png)


## 总结

对于一个小型的创业企业，如果你的程序员稍微能懂点 Linux 操作，一个月租一台阿里云最恰当的性价比的机器，就可以拥有一个不错的工作体验，我觉得性价比还可以。