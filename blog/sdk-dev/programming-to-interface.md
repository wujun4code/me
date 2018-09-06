---
date: "2018-02-20"
title: "SDK 开发的经验分享 - 面向接口/协议编程"
category: "Coding"
---
<!-- TOC -->

- [面向接口/协议编程](#面向接口协议编程)
    - [我需要一个 Http Client 来帮助我收发 Http 请求](#我需要一个-http-client-来帮助我收发-http-请求)
    - [抽象出有用的相同的，封装掉用不到和不同的](#抽象出有用的相同的封装掉用不到和不同的)
    - [封装一个 IHttpClient，它只需要做好一件事情](#封装一个-ihttpclient它只需要做好一件事情)
    - [<a name="begin-to-use-httpclient"></a>开始消费 Http Client](#a-namebegin-to-use-httpclienta开始消费-http-client)

<!-- /TOC -->
# 面向接口/协议编程

这个话题真的是老生常谈，烂大街的面试口号了，但是时至今日，依然可以看见众多开源的项目里面，分层逻辑根本没有遵照面向接口/协议编程。

并不是定义了一个接口，你用类去实现了它，但是你通篇代码都是在 new 这个类，然后到处依赖这个类，这就叫面向接口/协议编程了。

我们从实际的 SDK 开发的需求来阐述这个命题。


## 我需要一个 Http Client 来帮助我收发 Http 请求

常规做法，在 Swift 中引用时下最流行的 Alamofire 作为我的包依赖，然后只要负责调用它就好了，没错，大家都这么想也都这么做，但是你直接在代码里面到处调用 Alamofire.request，你有没有想过，万一 Alamofire 停更了怎么办？万一你发现性能更好的替代品怎么办？还有，使用它，就必须遵循它的一些特性和去适配一些它独特的用法， 这些都是在变化的，面对对象开发的过程中，最忌讳的就是你对变化一如既往的认为它「应该」不会变吧。

## 抽象出有用的相同的，封装掉用不到和不同的

首先，我们来看看一个 Http 请求在 SDK 级别需要使用到的属性：

- Http Headers 

  现在大多数的云服务或者说哪怕是自建的云服务，都会在 Http Headers 里面存放一些统计或者是鉴权数据，例如客户端版本，用户标识，还有客户端的 Cookie 之类的，因此它就是一个通用的属性，我们应该在接口里面暴露出来

- Http Method

  众所周知，现在绝大多数厂商提供的都是基于 REST 风格的 API，而 REST 风格的本意就是使用 Http Method 的几个不同取值（POST/DELETE/PUT/GET），来告知服务端，我是来 增/删/改/查的，因此它也是需要在接口里面暴露出来的。

- Http Body

  在 POST 和 PUT 请求中，必须包含一个 Http Body，否则服务端都不知道你要新增和修改是什么。

- Http Url

  这个就不用说了，一个 Http 请求的送达的地址，是一定需要暴露给 SDK 去设置的。


SDK 对数据的操作无非就是增删改查，那么对于一个使用 Http 协议消费 REST API 的 SDK来说，它只需要对 Http 请求上的上述四个属性进行操作，就可以满足增删改查，那么剩下那些不需要的东西，就应该向 SDK 透明掉。例如 Alamofire 有一些比较独特编码模式这些就是属于，SDK 不需要去知道的，我只需要你来帮我把我感兴趣的四个属性，当做一个 Http 请求给我发出去就好了。

因此我们在 SDK 内部基于当前的业务逻辑重新定义了一下 Http Request。

```swift
public class HttpRequest {
    var method: String
    var url: String
    var headers: Dictionary<String, String>?
    var data: Data?

    init(method: String, 
         url: String, 
         headers: Dictionary<String, String>?, 
         data: Data?) {
        self.url = url
        self.headers = headers
        self.data = data
        self.method = method
    }
    convenience init(method: String, 
                     url: String, 
                     headers: Dictionary<String, String>?, 
                     jsonData: Dictionary<String, Any>?) {
        let data = jsonData?.binarization()
        self.init(method: method, url: url, headers: headers, data: data)
    }
}
```
```typescript
export class HttpRequest {
    public url: string;
    public headers: { [key: string]: string };
    public data: { [key: string]: any };
    public method: string;

    constructor() { };
}
```
```java
public class HttpRequest {
    public String url;
    public String method;
    public InputStream body;
    public Map<String, String> headers;

    public String  getContentType() {
        return this.headers.get("Content-Type");
    }

    public String getUrl() {
        return url;
    }

    public InputStream getBody() {
        return body;
    }
}
```


*注意*：在真正的 Http 请求里面保罗万象，比如 keep-alive 等特性，实际上在 SDK 中并不需要关心这个，它只需要确保自己选择第三方 Http Client 库能实现这些基本的功能即可。这就是我们上面说的向 SDK 透明掉用不到的。

## 封装一个 IHttpClient，它只需要做好一件事情

上面的 HttpRequest 已经就绪，此时 SDK 需要一个工具来发送它，代码如下：

```swift
public class HttpClient {
    public func rxExecute(httpRequest: HttpRequest) -> Observable<HttpResponse> {

    }
}
```
```typescript
export class RxHttpClient {
    execute(httpRequest: HttpRequest): Observable<HttpResponse> {

    }
}
```
```java
public class JavaHttpClient {
    HttpResponse execute(HttpRequest request) {

    }
}
// Java 默认支持的是在当前线程执行 Http 请求，如何在 Android 里面实现异步，那是另一个话题->关于异步的封装
```


上面我新建了一个具体的类，来实现发送 HttpRequest 的需求，但是我们知道，类是不稳定的，类是异变的，因此我们把这个类的行为抽象出来，成为一个接口，然后让这个类去实现它，而 SDK 全局只要使用这个接口就可以了，哪怕后来者有人想修改替换这个类，只要他重新提供一个实现了这个接口的类给 SDK，就能保证 SDK 继续健康的工作。

```swift
import Foundation
import RxSwift
public protocol IHttpClient {
    func rxExecute(httpRequest: HttpRequest) -> Observable<HttpResponse>
}
```

```typescript
import { Observable } from 'rxjs';
import { HttpRequest } from './HttpRequest';
import { HttpResponse } from './HttpResponse';
export interface IRxHttpClient {
    execute(httpRequest: HttpRequest): Observable<HttpResponse>;
}
```

```java
public interface IHttpClient {
    HttpResponse execute(HttpRequest request);
}
// Java 默认支持的是在当前线程执行 Http 请求，如何在 Android 里面实现异步，那是另一个话题->关于异步的封装
```

完整的代码如下：

```swift
public class AlamofireHttpClient: IHttpClient {

    open static let `default`: AlamofireHttpClient = {
        return AlamofireHttpClient()
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
```typescript
export class AxiosRxHttpClient implements IRxHttpClient {

    execute(httpRequest: HttpRequest): Observable<HttpResponse> {
        let tuple: [number, any] = [200, ''];
        return Observable.fromPromise(this.RxExecuteAxios(httpRequest)).map(res => {

            tuple[0] = res.status;
            tuple[1] = res.data;
            let response = new HttpResponse(tuple);
            RxAVClient.printLog('Response:', JSON.stringify(response));
            return response;
        }).catch((err: any) => {
            RxAVClient.printLog('Meta Error:', err);
            return Observable.throw(err);
        });
    }

    RxExecuteAxios(httpRequest: HttpRequest): Promise<AxiosResponse> {
        let method = httpRequest.method.toUpperCase();
        let useData = false;
        if (method == 'PUT' || 'POST') {
            useData = true;
        }
        return new Promise<AxiosResponse>((resovle, reject) => {
            axios({
                method: method,
                url: httpRequest.url,
                data: useData ? httpRequest.data : null,
                headers: httpRequest.headers
            }).then(response => {
                resovle(response);
            }).catch(error => {
                reject(error);
            });
        });

    }
}
```
```java
public class RxLeanCloudJavaGeneralHttpClient implements IHttpClient {
    private static RxLeanCloudJavaGeneralHttpClient ourInstance = new RxLeanCloudJavaGeneralHttpClient();

    public static RxLeanCloudJavaGeneralHttpClient getInstance() {
        return ourInstance;
    }

    OkHttpClient client = new OkHttpClient();

    @Override
    public HttpResponse execute(HttpRequest request) throws IOException {
        Request.Builder builder = new Request.Builder();
        MediaType CONTENT_TYPE = MediaType.parse(String.format("%s; charset=utf-8", request.getContentType()));
        switch (request.method.toUpperCase()) {
            case "GET": {
                builder.get();
                break;
            }
            case "DELETE": {
                builder.delete();
                break;
            }
            case "PUT": {
                try {
                    builder.put(RequestBody.create(CONTENT_TYPE, this.readBytes(request.getBody())));
                } catch (IOException e) {
                    e.printStackTrace();
                }
                break;
            }
            case "POST": {
                try {
                    builder.post(RequestBody.create(CONTENT_TYPE, this.readBytes(request.getBody())));
                } catch (IOException e) {
                    e.printStackTrace();
                }
                break;
            }
            default: {
                builder.get();
            }
        }
        request.headers.forEach((k, v) -> builder.addHeader(k, v));
        Request okHttpRequest = builder.url(request.getUrl()).build();
        Response okHttpResponse = this.client.newCall(okHttpRequest).execute();

        HttpResponse response = new HttpResponse();
        response.statusCode = okHttpResponse.code();
        response.body = okHttpResponse.body().bytes();

        return response;
    }

    public byte[] readBytes(InputStream inputStream) throws IOException {
        byte[] b = new byte[4096];
        ByteArrayOutputStream stream = new ByteArrayOutputStream();
        int c;
        while ((c = inputStream.read(b)) != -1) {
            stream.write(b, 0, c);
        }
        stream.flush();
        byte[] result = stream.toByteArray();
        return result;
    }
}
```

## <a name="begin-to-use-httpclient"></a>开始消费 Http Client

首先，我们给我们的 SDK 设置一个插件库，它负责存储 Http Client 的对象，SDK 要用的时候就去这个插件库里面取出来就好了，SDK 内有许多对象都需要去获取这个 Http Client，插件库代码如下：

```swift
public class AVCorePlugins {
    static let sharedInstance = AVCorePlugins()

    private var _httpClient: IHttpClient = AlamofireHttpClient.default as IHttpClient
    var httpClient: IHttpClient {
        get {
            return self._httpClient
        }
    }
}
```

```typescript
export class SDKPlugins {

    private static _sdkPluginsInstance: SDKPlugins;
    static get instance(): SDKPlugins {
        if (SDKPlugins._sdkPluginsInstance == null)
            SDKPlugins._sdkPluginsInstance = new SDKPlugins(1);
        return SDKPlugins._sdkPluginsInstance;
    }

    private _HttpClient: IRxHttpClient;
    get HttpClient() {
        if (this._HttpClient == null) {
            this._HttpClient = new AxiosRxHttpClient();
        }
        return this._HttpClient;
    }
}
```
```java
public class RxAVCorePlugins {
    private static RxAVCorePlugins ourInstance = new RxAVCorePlugins();

    public static RxAVCorePlugins getInstance() {
        return ourInstance;
    }

    private RxAVCorePlugins() {
    }

    private IHttpClient httpClient;

    public void setHttpClient(IHttpClient httpClient) {
        this.httpClient = httpClient;
    }

    public IHttpClient getHttpClient() {
        return httpClient;
    }
}
```

当时走到这一步，我需要先卖一个关子，并不是直接从 Plugins.HttpClient 才是最正确的使用方式，这里牵涉到另一个问题：依赖注入


