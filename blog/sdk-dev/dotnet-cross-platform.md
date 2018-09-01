---
date: "2015-10-26"
title: "SDK 开发的经验分享 - 跨平台共享接口"
category: "Coding"
---

# 哪里都缺程序员

“老王，我这里钱有了，产品概念有了，就差一个程序员了，你 iOS ， android ， Windows 你都会么？”

“我……额，喂！信号不好，我这里听不见……”
“嘟……”

其实，很多产品初创的时候，做一个应用不必考虑一定要追求极致的性能，都奔着 Native Code 去做，大火大热的 React Native 风生水起，其实 Cross Platform 可以作为一个初创团队人手不足的情况下应该考虑采用这种跨平台的去实现自己的产品的前几个迭代版本，毕竟人是活的，代码是死的。

# 跨平台的玩具 - Xamarin （现已经被微软收购，即将推出历史舞台）

因为笔者本身是 .NET 出身，也关注 Xamarin 很久了，自从微软把他收编之后我发现其实，一套代码做出 iOS ， Android ， Windows （ WinForm & WPF ）， Windows Phone 这件事情在实现上成为可能。

不过随着开发的深入，你会发现，坑还是有，并且很深。诸多 Xamarin 的教程我就不列举了。我只是来简述一下如何踩坑。

最简单的一个需求：我要在各个平台上实现一个微信的聊天服务，服务端已经有了使用 WebSocket 现成的服务，而客户端需要有一个 WebSocket 客户端去实现收发数据，那么如何用高度一致的代码在

Xamarin iOS
Xamarin Andoid
Windows PC
Windows Phone

实现这样的需求呢？

老思路解决新问题
面对对象的小蛮腰就是多态，迷人的变化。
在刚才列举的四个运行时下，除了 Windows PC 下有微软官方实现的一个WebSocket for Windows PC runtime，其余的都没有官方给出实现，因此我们会想到，在 Windows Phone | Xamarin Andoid | Xamarin Andoid 分别采用第三方库来解决这个问题，现在问题来了，本来我的想法是一套代码解决这些问题，可是一旦引入了第三方库，接口就会不一样，我就要针对每一个平台单独去编写收发消息的方法，例如下面这种是很多程序员经常使用的方式：

```cs
#if Xamarin_Andoid || Xamarin_iOS|| Windows_Phone
#endif

#if Windows_PC
#endif
```
这种方式实现多平台共享代码是很痛苦的，并且这种实现方式的上下逻辑得非常高明，这个对程序员的脑子烧的很痛。

## 接口的引入——开始「变态」 了
因此，我们需要封装掉变化，引入一个接口，让每个平台的底层实现向上透明，我只告诉你我的接口可以实现收发消息，至于我底层采用什么库，用什么框架你别管。

```cs
public interface IWebSocket
    {
        /// <summary>
        /// 初始化 WebSocket 连接
        /// </summary>
        /// <param name="server">服务器地址.</param>
        /// <returns>返回的是一对值，状态码和具体的返回值</returns>
        Task<Tuple<int, string>> InitWebSocket(string server);

        /// <summary>
        /// 异步发送(基于 Task 的编程已经是 .NET 的主流了.)
        /// </summary>
        /// <param name="data">The data.</param>
        /// <returns>int 表示状态码，参考 HTTP StatusCode 设计的，而 String 可能是服务器告诉你这条消息发送之后的一些参数，比如收到的具体时间，等等都编码之后封装在 String 里面，客户端自行解析</returns>
        Task<Tuple<int, string>> SendAsync(string data);

        /// <summary>
        /// 定义收到消息之后激发的事件,这是聊天应用常用的被动通知
        /// </summary>
        event EventHandler<string> OnReceived;

        /// <summary>
        /// 当前连接的状态,可能断线了,可能尚未初始化.
        /// 这个枚举其实每个库都会有自己定义的类型,但是我依然在我自己的应用中定义了符合自己需求的
        /// </summary>
        /// <value>
        /// 连接状态
        /// </value>
        SocketStatus status { get; set; }
    }
    public enum SocketStatus
    {
        None,
        SocketConnected,//socket opend.
        SocketDisconnected,//socket closed.
    }
```

然后呢，每一个项目里面单独实现这个接口：

```cs
namespace MyProject.RealtimeMessage
 {

 public class MyWebSocket : IWebSocket{
 // 接口实现的一些代码
 }

 }
 ```
这里有一个必须声明的地方，这个实现的接口的类的名字必须一致（每个项目都有这个类，都叫 MyWebSocket）。

然后在真正初始化的时候，我们需要如下代码在运行时把他召唤出来：

```cs
Type webSocketType = Type.GetType("MyProject.RealtimeMessage.MyWebSocket");

IWebSocket webSocket = Activator.CreateInstance(webSocketType) as IWebSocket;
```

这样每一个平台使用 IWebSocket 这个接口的功能去具体实现自己收发消息的功能，而且逻辑代码是每个项目(csproj)都可以共用的（用快捷方式引入代码这个技巧不用我说吧……）

以上只是抽象了我从真实项目里面实践的经验简单的描述了一下。

另外，更高级的做法是，你可以允许开发者自己来实现这个接口，然后在运行时，你给一个地方，让他把实现了这个 `IWebSocket` 的对象实例传递给你，这样开发者的可控度会更高，开发体验会更好，他可以尽情的在这个对象里面做一些日志追踪打印之类的事情，而 SDK 本身只要调用接口规定的几个方法就可以了。

## 小结
这种操作方式，可以在 Java/JavaScript/Typescript 这些能在各种环境下运行的语言，这只是一个常规操作的技巧，其本身并没有多少玄幻的东西，唯一的难点是准确的定义好这个接口，并且一定要保证，在后期迭代的过程中千万不要轻易修改它，否则代价就是你得所有平台都得重新改，重新测试，重新发布。

