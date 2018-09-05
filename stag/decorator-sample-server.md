---
date: "2018-09-05"
title: "业务和资源的分离 - 装饰器模式的一次尝试"
category: "Coding"
---
<!-- TOC -->

- [业务逻辑服务器和前端链接服务器的拆分](#业务逻辑服务器和前端链接服务器的拆分)
- [基于 ASP.NET Core 的 WebSocket 协议分层改造](#基于-aspnet-core-的-websocket-协议分层改造)

<!-- /TOC -->

## 业务逻辑服务器和前端链接服务器的拆分

当前大部分（绝大部分）的服务端的架构都将业务逻辑的服务器和前端链接（长短链接都一样）拆开，比如经典的 LAMP 架构。

本文的源头是在于我看了许多游戏服务器的源码，很多设计都是比较棒的分层，比如真正的游戏服务端逻辑用 Lua 编写，而通过 Lua 去调用 C++/Go 编译好的方法，实现了分层，但是我注意到一个细节，这些服务器在更靠前的逻辑：长连接管理层依然耦合在核心的游戏的 C++/Go 的代码里面：

> 此处的耦合指的是包括 Socket 管理，Tcp/Udp/WebSocket 的协议都在这一层

而按照我个人的理解，先扔掉 Socket 这个一层偏向于应用层的管理不说，协议这一层应该被拆成一个独立的服务，我们可以这样理解：

> Tcp/Udp/WebSocket 假设是一个开箱即用的服务，那么不管后面是游戏服务器，聊天服务器，推送服务器，还是最常用的 Web 服务器，都可以藏在这个服务后面，客户端消费的时候，只要用目标协议连上这个服务，而背后的业务层订阅对应的协议上的链接变化，来做对应的处理，这样设计可能会舒服一些。


## 基于 ASP.NET Core 的 WebSocket 协议分层改造

```sequence
Andrew->China: Says Hello
Note right of China: China thinks\nabout it
China-->Andrew: How are you?
Andrew->>China: I am good thanks!
```
