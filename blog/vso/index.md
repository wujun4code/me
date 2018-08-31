---
date: "2018-08-31"
title: "Visual Studio Online CI 使用"
category: "CI"
---

<!-- TOC -->

- [适用人群](#适用人群)
- [注册和登录以及初始化](#注册和登录以及初始化)
- [编写第一个 Build](#编写第一个-build)
    - [创建一个 Build](#创建一个-build)
    - [编译之后发布到 Github Release](#编译之后发布到-github-release)
    - [只写代码，master 合并之后自动发布](#只写代码master-合并之后自动发布)
- [使用自定义脚本可以做任何事情](#使用自定义脚本可以做任何事情)

<!-- /TOC -->

## 适用人群

- 你是程序员/测试/SA/QA/产品经理
- 你不习惯 Travis CI 的龟速和受够了 Jenkins 的 UI
- 你需要友好的 UI 操作来制定测试/发布流程
- 你对微软的产品不排斥
- 你的代码是跨平台的编译到各种环境下


## 注册和登录以及初始化
访问 [https://visualstudio.microsoft.com/](https://visualstudio.microsoft.com/) 注册或者直接登录你的微软账号(hotmail/outlook邮箱都可以)。

然后你会看见如下界面：

![image](https://user-images.githubusercontent.com/5119542/44890228-0b900200-ad0c-11e8-91de-e3a3b106193d.png)

接下来你需要创建一个组织

- 组织里面包含了多个项目 - 对应的就是 Github 多个 repo
- 每一个项目都可以创建多个自动化测试的 Build 和 Release
- 每一个组织默认免费可以有 5 个成员（包括创建者自身）和 240 分钟的云端 CI 运行时间

创建组织的时候输入一些必要的字段，比如二级域名，`myapp.visualstudio.com` 这个就是你以后访问的组织域名。

![image](https://user-images.githubusercontent.com/5119542/44890510-2adb5f00-ad0d-11e8-938b-f76018ae9472.png)


接着需要创建一个项目 `MyBlog`:

![image](https://user-images.githubusercontent.com/5119542/44890584-74c44500-ad0d-11e8-85ce-857db5aa21e7.png)

## 编写第一个 Build
首先，从我自身的项目出发[https://github.com/leancloud/realtime-SDK-dotNET](https://github.com/leancloud/realtime-SDK-dotNET) ,它是 LeanCloud 实时通信 SDK C# 源码，我每次发布新版 SDK 都是从 VSO 里面发布。

### 创建一个 Build

第一步，点击 `New Pipeline`:

![image](https://user-images.githubusercontent.com/5119542/44911070-1e302880-ad58-11e8-951b-02f71773f481.png)

第二步，选择你代码所在的托管平台:

![image](https://user-images.githubusercontent.com/5119542/44911014-df01d780-ad57-11e8-9671-c490c5698417.png)

如果选择 Github 就会开始发起 Github 的 OAuth 授权，授权完毕之后，如图选择 `use the visual designer`：

![image](https://user-images.githubusercontent.com/5119542/44911831-d363e000-ad5a-11e8-9669-ed7add643a9e.png)

选择默认的 .NET Desktop 模板:

![image](https://user-images.githubusercontent.com/5119542/44911879-0ad28c80-ad5b-11e8-9b3d-218a8a429d3f.png)

现在应该很清楚了，实际上所谓的 CI 也就是一步一步的脚本帮你实现了你原本应该在本地做的事情，然后我们需要修改它直到它的编译过程符合你的需求:

![image](https://user-images.githubusercontent.com/5119542/44912073-ac59de00-ad5b-11e8-9479-8a414361eb35.png)

请原谅我没有一步一步的重写，我直接照搬了我线上的步骤，这不是重点。

### 编译之后发布到 Github Release

先看效果图：

![image](https://user-images.githubusercontent.com/5119542/44912189-f4790080-ad5b-11e8-9cda-13ee775d4027.png)

这上面的 SDK 发布都是通过刚才那个 Build 自动发布的。

我们主要看最后那个关键的 Github Release:

![image](https://user-images.githubusercontent.com/5119542/44912245-212d1800-ad5c-11e8-9061-6501f6ba21fe.png)

它是一位大神编写第三方脚本，但是超级好用。

它整个步骤也比较清楚，就是直接从你的文件夹目录中拷贝文件(dll/xml 或者任何其他属于 SDK 内容的文件)发布到 [https://github.com/leancloud/realtime-SDK-dotNET/releases](https://github.com/leancloud/realtime-SDK-dotNET/releases) 里面去。


### 只写代码，master 合并之后自动发布

![image](https://user-images.githubusercontent.com/5119542/44912393-8f71da80-ad5c-11e8-806b-f8dcfbf522ad.png)

按照上述这是一个 Trigger 当主分支有代码合并进来，这个 Build 就会自动开始跑，一旦跑成功就会自动发布到 [https://github.com/leancloud/realtime-SDK-dotNET/releases](https://github.com/leancloud/realtime-SDK-dotNET/releases) 。


## 使用自定义脚本可以做任何事情

![image](https://user-images.githubusercontent.com/5119542/44912465-cc3dd180-ad5c-11e8-9df8-bd05447b7a4b.png)

添加一个步骤的时候，你可以搜索和选择你想要的类型，这里面几乎全面支持各种 shell 和各种命令行工具。