---
date: "2018-11-12"
title: "SDK 开发的经验分享 - 委托、事件、异步，多线程的纷杂"
category: "Coding"
---


## 委托和事件

聊天其实就是即时通讯或者说消息转发云服务的 SDK 有一个比较重要的一环：

> 被动的事件监听 API 是需要精心设计

一般会采用如下设计：

```cs
public event EventHandler<MessageEventArgs> OnMessageReceived;
```

基于事件实现接收消息的通知是 「最理所应当」的方式，

而事件本质就是一个委托：

> 诸葛亮给赵云三个锦囊救刘备，这三个锦囊就是三个委托，而每次遇到事件触发，赵云就打开对应的锦囊，这就是三国世界里面的委托。

也就是因为委托可以如下关键需求：

1. 我先预设一下假设我收到了消息之后怎么处理的业务函数
2. 一旦收到了消息，直接执行我预设的业务函数


## 异步和多线程

写带 UI 的客户端软件或者应用的时候，普遍需要处理一种情景：

> 我输入了用户名和密码之后，点击登录按钮，此时 UI 线程必须被释放，真正启动一个子线程去等待服务端的返回结果，而另外 UI 线程会订阅这个子线程，因为这个子线程会将服务端的校验结果返回给 UI 线程，UI 线程根据登录成功或者是密码错误做出对应的页面跳转或者是错误提示。


这个整个过程就是异步和多线程的经典用例。

而这个用例的解决方案恰恰可以用到事件和委托：


登录的业务逻辑：

```cs
public event EventHandler<LogInEventArgs> OnLogInResult;
public void LogIn(string username,string password)
{
    new Thread(()=>
    {
        // 这里使用了经典的回调，这本质上也是个委托
        SendRequest("LogIn", username, password, result =>
        {
            OnLogInResult.Invoke(this,result);
        });
    });
}
```

UI 逻辑:

```cs
OnLogInResult += (sender,args) =>
{
    RunOnUIThread(()=>
    {
        if(result.Error == null)
        {
            // 登录成功，跳转到主页
        }
        else 
        {
            // 弹出窗口提示服务端返回的错误信息
            ShowMessage("登录失败",result.Error.Message);
        }
    });
};
```

## 事件/委托和异步/多线程的混淆点

此时结合 `OnMessageReceived` 和 `OnLogInResult` 我们会发现:

> 主动的登录操作和被动的消息接收都可以用事件来解决

但是如果我新增一个注册操作呢？新增一个发消息的操作呢？新增一个监听网络断开的事件呢？

于是乎，你就会发现：

> 当我们把所有的异步都依赖事件来实现的时候，每次新增一个异步，你就需要新增一个事件或者修改原事件的参数，这严重影响到了代码的稳定性，并且新来的同事看你的代码会骂娘。

那么有没有更优雅的解决方案呢？

## async/await 的呼之欲出

首先我们必须严肃一个观点：

> 主动的异步操作和被动的事件通知，虽然本质上都可以视为是委托和异步的场景，但是我们往往需要区分主动和被动的行为结果。


首先我们把主动的异步操作改为如下:

```cs
public async Task<LogInEventArgs> LogInAsync(string username,string password);
```

这样我们就明确区分了主动和被动的两种实现，这实际上是现在被证明也是被广泛使用的一种编程范式，大家都这么干，没错，但是作为有编程洁癖的程序员肯定就会想，既然主动异步和被动事件通知曾经可以使用同一种解决方案，现在使用了 async/await，让他们现在彼此分裂，那么现在是不是还有方法，让他们俩能够重新归一统呢？

答案：必须有

请看我之前的博客:[初探 Rx](初探-rx)