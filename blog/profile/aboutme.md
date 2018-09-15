---
date: "2018-09-16"
title: "关于我"
category: "Profile"
---


# 联系方式

- 手机/微信号：416318965：18612438929
- Email：wujun19890209@gmail.com
- QQ：416318965

---

# 个人信息

 - 吴骏/男/1989 
 - 本科/辽宁科技大学/软件工程
 - 工作年限：7年
 - 微博：[@神笔胖码农](http://weibo.com/416318965) 
 - 技术博客：http://wujun.blog
 - Github：http://github.com/wujun4code

 - 期望职位：软件结构设计师，客户端软件架构
 - 期望薪资：税前月薪28k，特别喜欢的公司可例外
 - 期望城市：苏州/上海

---

# 工作经历


## LeanCloud （ 2014 年 5 月 ~ 至今 ）

### 产品文档项目
负责全站所有产品面向开发人员的技术文档编写和维护，包括文字和示例代码，文案结构，章节顺序全权负责。在这个项目里面额外的增加了一些 jQuery/Angular 等前端的一些工作，项目中最困难的部分是多语言示例代码的维护，最后我采取了 nunjunks 模板渲染的宏来解决将各语言的实例代码整合到一个 md 文件里面，直接减少了五分之四的文件数量。最让我感受到比较重要的成果是重新认识了一些前端的工具，包括 grunt/gulp/scss 还有 jQuery/Angular 相关的前端技术栈，自己搭建一个小的 web 站点应该难度不大。


### C# SDK 研发和维护
负责 LeanCloud 云服务中云存储，即时通讯，LiveQuery 等功能模块的 C# SDK 的研发和维护，其中比较核心的是 Unity 平台的适配，遇到了太多 Unity 与 C# 的版本不兼容的问题，用了预编译符号和抽象接口的方式将 Unity 和 C# （包括服务端 .NET Core）也通过同一份代码编译出不同平台均可使用的 SDK。主要是在封装接口上花了一些时间研究和维护，另外就是对 Unity 的游戏编程也有一点了解，从 Unity 4.6 一直更新到 5.5，踩过坑太多，但是也觉得 Unity 是个不错的引擎。 


### 其他项目

- 游戏云服务，类似于一个实时对战的即时通讯的 SDK，面向 Unity 的，写了第一版之后因为有新同事接手，就没继续参与。
- 产品介绍页的设计，用静态 html/css 设计了即时通讯新版产品介绍页。
- RxLeanCloud，自己业余时间维护了 Swift/Java/TypeScript 针对 LeanCloud 云存储和即时通讯的 SDK，基于 Rx 风格全面重写，地址:[RxLeanCloud](https://github.com/rxleancloud)。

 
## 上海微创（北京分公司） （ 2013 年 10 月 ~ 2014 年 4 月 ）

### CloudBox 服务端 
基于 System Center 的 Powershell API 封装成 WCF/REST api 给客户端调用，来实现企业虚拟桌面的主机创建，云资源管理，子网监控等等，主要客户都是一些银行企业，大型企业的虚拟桌面化办公，对应的产品是 VMWare。过程中接触到了微软 Azure 的公有云和混合云，也深入了解了 Azure 的 REST API，其中印象比较深刻的是异步的 REST API 的封装，因为创建虚拟机是一个异步的过程，因此在封装 API 的时候需要也给出一个异步的解决方案，最后成功上线，还拿下了电信的项目，是花费精力最少，性价比最高的一个项目了。

### 其他项目

- 在内部做了一些 REST API 的培训

## Symbio 北京 （ 2011 年 10 月 ~ 2013 年 7 月 ）

### Nokia App Studio
为 Nokia 收购版权的 iOS 应用做出对应的 Windows Phone 的版本，主要负责了 3 个游戏，使用到了末代的 XNA 引擎和 Silverlight，开始逐步了解微软的 XAML 技术栈，对这个至今都觉得很棒，这是 MVVM 结构的一个很棒的实现，对依赖属性，对 XAML 的文件编写都很感兴趣，当时是初学者，项目比较大，我们当时五六个人一起做了二十多个游戏，每个人都要负责三四个。

### 其他项目

- EverNote 的 WP 版本维护
- 微软创投加速器的官网开发


# 开源项目和作品

## 开源项目

 - [RxLeanCloud-Swift](https://github.com/RxLeanCloud/rx-lean-swift)
   基于 Swift 4.0 全新编写 LeanCloud 客户端 SDK
 - [RxLeanCloud-Java](https://github.com/RxLeanCloud/rx-lean-java)
   基于 Java 全新编写 LeanCloud 客户端 SDK
 - [RxLeanCloud-TypeScript](https://github.com/RxLeanCloud/rx-lean-js-core)
   基于 TypeScript 全新编写 LeanCloud 客户端 SDK

 - [DoChat](https://github.com/wujun4code/DoChat)
   一个基础 ionic 的聊天应用 demo

 - [LeanCloud Docs](https://github.com/leancloud/docs)
 - [LeanCloud realtime-SDK-dotNET](https://github.com/leancloud/realtime-SDK-dotNET)
 - [LeanCloud leanengine-dotNET-sdk](https://github.com/leancloud/leanengine-dotNET-sdk)


## 技术文章
都在博客:[https://wujun.blog](https://wujun.blog) 整理和发布


# 技能清单


以下均为我熟练使用的技能

- Web 开发：ASP.NET/Node/jQuery
- Web 框架：ASP.NET MVC
- 前端框架：Bootstrap/Angular/ionic
- 前端工具：Grunt/Gulp/SaSS/
- 数据库相关：SQL Server/SQLite/MongoDb
- 版本管理、文档和自动化部署工具：git/Gatsby/Travis CI/VSO CI
- 单元测试：mocha/XUnit/NUnit
- 云和开放平台：SAE/Azure/AWS/微博开放平台/微信应用开发/微信公众号/微信企业号

## 参考技能关键字

- C#
- Angular
- Unity3D/NGUI
- WPF/Silverlight
- .NET Core
- ASP.NET MVC
- ionic
- Xamarin
- Azure
- TypeScript
- Docker/Compose
- Markdown
- Visual Studio/Visual Studio Code
- RocketChat/Slack
  

---

