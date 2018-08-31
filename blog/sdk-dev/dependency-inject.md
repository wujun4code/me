---
date: "2018-02-25"
title: "SDK 开发的经验分享 - 依赖注入"
category: "Coding"
---

<!-- TOC -->

- [问题的产生](#%E9%97%AE%E9%A2%98%E7%9A%84%E4%BA%A7%E7%94%9F)
- [解决方案](#%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88)

<!-- /TOC -->

## 问题的产生

依赖注入是一个稍微高端一点的面试都会浅尝辄止谈到的话题，大多数情况下，程序员会天然地以为这是「过度设计」，其实，在我看来，「过度设计」其实就是政治正确，政治正确只是一个程序员在写代码的时候最基本的编程信仰是否有洁癖，其实依赖注入是最基本的面对对象程序的「入门设计」，在代码中适当的使用，可以让你的代码在别人眼里获得更高的认同感，而不是说纯粹为了炫技或者说是为了政治正确而去使用依赖注入。

还是以我们前文说的 `HttpClient` 为例。

我们的 SDK 中已经拥有一个 HttpClient 的实例，并且它实现了 IHttpClient 的接口，新建一个对象的 POST 请求如下：

```bash
curl -X POST \
  -H "X-LC-Id: x7WmVG0x63V6u8MCYM8qxKo8-gzGzoHsz" \
  -H "X-LC-Key: PcDNOjiEpYc0DTz2E9kb5fvu" \
  -H "Content-Type: application/json" \
  -d '{"foo": "bar"}' \
  https://api.leancloud.cn/1.1/classes/Todo
```

换做是入行第一年的我会写出如下的代码来使用之前封装好的 `HttpClient`

```csharp
public class AVObject {
    public bool save() {
        IHttpClient httpClient = RxAVCorePlugins.getInstance().getHttpClient();
        Map<String,Object> encoded = this.encode();
        return httpClient.execute(encoded);
    }
}
```

上述代码实际上也不会有问题，只是在成熟的程序员眼里，这个写法有点太初级，因为这样写有一点点违反了面对对象设计里面的「低耦合」原则，AVObject 内部极度依赖 `CorePlugins` 里面的 `IHttpClient`，同时也会造成一种强制反向依赖，在这个场景下,

> 明明是消费者（AVObject）去消费插件（CorePlugins），而现在看上去一旦插件丢了（CorePlugins = null），那么消费者（AVObject）将无法继续消费（AVObject.save）

解决依赖问题在面对对象里面有一个最最基础的解决方案，那就是 A 依赖了 B 的实例，那么请在构造 A 的时候，顺便把一个不为空的 B 的实例，传递给 A 的构造函数，这样 A 在之后自己的业务逻辑里面要用的 B 的时候，不用再去临时去构建一个 B 的实例或者去获取 B 的单例，只需要访问自己在构造的时候传递进来的 B 的实例，即可，简而言之就是「依赖注入」。

## 解决方案

```swift
public class AVHttpCommandRunner: IAVCommandRunner {

    var httpClient: IHttpClient
    init(httpClient: IHttpClient) {
        self.httpClient = httpClient
    }

    public func runRxCommand(command: AVCommand) -> Observable<AVCommandResponse> {
        return command.beforeExecute().flatMap({ (cmd) -> Observable<HttpResponse> in
            return self.httpClient.rxExecute(httpRequest: cmd)
        }).map({ (httpResponse) -> AVCommandResponse in
            let avResponse = AVCommandResponse(statusCode: httpResponse.satusCode, data: httpResponse.data)
            return avResponse
        })
    }
}
```
```typescript
export class AVCommandRunner implements IAVCommandRunner {

    private _IRxHttpClient: IRxHttpClient;

    constructor(rxHttpClient: IRxHttpClient) {
        this._IRxHttpClient = rxHttpClient;
    }

    runRxCommand(command: AVCommand): Observable<AVCommandResponse> {
        return this._IRxHttpClient.execute(command).map(res => {
            return new AVCommandResponse(res);
        }).catch((errorRes) => {
            return Observable.throw(errorRes);
        });
    }
}
```
```java
public class AVHttpCommandRunner implements IAVCommandRunner {
    private IHttpClient httpClient;

    public AVHttpCommandRunner(IHttpClient httpClient) {
        this.httpClient = httpClient;
    }

    @Override
    public AVCommandResponse execute(AVCommand command) throws IOException, RxAVException {
        HttpResponse httpResponse = this.httpClient.execute(command);
        AVCommandResponse response = new AVCommandResponse(httpResponse);
        RxAVClient.getInstance().commandLog(command, response);
        if (response.statusCode > 201 || response.statusCode < 200) {
            throw new RxAVException(response.statusCode, response.errorMessage());
        }
        return response;
    }
}
```

当然上述代码有没有针对构造函数里面 `httpClient` 进行空值检查，这不是我们的重点（2333），重点是，这样的代码解决了对于一个全局静态变量的依赖，这一点不论在写 SDK 还是在写 SaaS 业务逻辑的时候，都应该遵守，毕竟商业项目里面，不只有你一个人会维护代码，需要时刻提醒自己如果需要依赖，请注入。

