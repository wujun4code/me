---
date: "2018-02-17"
title: "SDK 开发的经验分享 - 基本理念"
category: "Coding"
---

# SDK 开发的经验分享

## 用户需要易懂的接口

首先，SDK 的接口**最好**满足如下几个条件：

- 每一个行为(方法/函数)对应的是语义明确的英文动词，例如 todo.save()
- 每一个属性（字段）对应的是语义明确的英文名词，例如 todo.objectId
- 通用的操作尽量使用 set/get/put 这种被广泛使用的单词，开发者容易从之前的经验中快速接纳你的设计
- 如无必要，减少对当前语言基础类型的封装（魔改）
- 面对对象的语言尽量减少静态函数的暴露以及到处泛滥的远程调用式的函数


## 可以容纳一切意外的输入

这里分为两种情况：

- 客户端用户的代码在运行时出现了不规范的类型插入和不规范的流程使用，切勿就抛出异常一了百了


```swift
public func set(key: String, value: Any?) {
    if value == nil {
        // 用户居然设置为 nil，那我就认为你是为了删除这个属性好了，请放心我不会抛出异常的。
        self.performOperation(key: key, operation: AVDeleteOperation.sharedInstance)
    } else {
        let valid = AVCorePlugins.sharedInstance.avEncoder.isValidType(value: value!)
        if valid {
            self.performOperation(key: key, operation: AVSetOperation(value: value!))
        }
    }
}
```

- 服务端返回了意外的格式，SDK 应该在设计的时候就假设服务端并不是永远都正常，返回了意外的格式甚至服务端无法访问，也应该能保证用户代码不会导致应用的崩溃，并且此时要做好日志打印和现场保留，以便确认问题的根本原因

```swift
public func httpLog(request: HttpRequest, response: HttpResponse) -> Void {
    if _enableLog {
        print("===HTTP-START===")
        print("===Request-START===")
        print("Url: ", request.url)
        print("Method: ", request.method)
        print("Headers: ", request.headers ?? ["no": "headers"])
        print("RequestBody(UTF8String): ", request.data?.stringfy(encoding: .utf8) ?? ["no": "body"])
        print("===Request-END===")
        print("===Response-START===")
        print("StatusCode: ", response.satusCode)
        print("RepsonseBody: ", response.bodyString)
        print("===Response-END===")
        print("===HTTP-END===")
    }
}
```

## 不惧怕过度设计，做到活用依赖注入，保持面向接口/协议编程基本的编程素养

比如现在开发 iOS SDK 常常会依赖 Alamofie 作为 HTTP Client 去收发请求，但是请在你的 SDK 里面对它进行封装，保证全局只有一个文件会出现对它的 import，这一点会在依赖注入的章节详细介绍。
与此类似的是 Java 开发中的 OkHttp 也需要 SDK 开发者对其进行透明化的接口封装。

面向接口/协议编程是烂大街的口号，但是诸多开源的 SDK 都没有很好的遵循这一点，简而言之，你依赖的一个函数，如果他来自于你依赖的一个接口，而不是来自于一个具体的对象，SDK 代码的安全感就多一层，反之就少一层，对象是易变的，接口是需要稳定的。

```swift
public protocol IHttpClient {
    func rxExecute(httpRequest: HttpRequest) -> Observable<HttpResponse>
}

public class HttpClient: IHttpClient {

    open static let `default`: HttpClient = {
        return HttpClient()
    }()

    public func rxExecute(httpRequest: HttpRequest) -> Observable<HttpResponse> {
        let manager = self.getAlamofireManager()
        let method = self.getAlamofireMethod(httpRequest: httpRequest)
        let urlEncoding = self.getAlamofireUrlEncoding(httpRequest: httpRequest)
        let escapedString = httpRequest.url.addingPercentEncoding(withAllowedCharacters: NSCharacterSet.urlQueryAllowed)

        return manager.rx.responseData(method, escapedString!, parameters: nil, encoding: urlEncoding, headers: httpRequest.headers).map { (response, data) -> HttpResponse in
            let httpResponse = HttpResponse(statusCode: response.statusCode, data: data)
            AVClient.sharedInstance.httpLog(request: httpRequest, response: httpResponse)
            return httpResponse
        }
    }
}
```

## 异步一定要优雅，Rx 最好，次一点也要 Task.Async ，再不济 Promise/Future 也行，抵制 Backbone 式的回调，从你我做起

如果还坚持使用 callback 和 block，那您出门左转，这里不适合你。

## 对用户的操作要警惕，用户随时可能反悔之前的操作

因此在客户端做好数据状态的管理，多存一份原始数据，在现如今的客户端设备上根本不会造成性能问题，详细请参考操作树。

