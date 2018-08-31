---
date: "2018-03-02"
title: "SDK 开发的经验分享 - 本地缓存"
category: "Coding"
---
<!-- TOC -->

- [本地缓存](#本地缓存)
    - [问题的来源](#问题的来源)
    - [问题的解决](#问题的解决)
        - [依赖注入](#依赖注入)

<!-- /TOC -->
# 本地缓存

## 问题的来源

Java 可以运行在云引擎也可以运行在 Android 以及全世界大概 150 亿的设备上，而 SDK 对本地缓存是刚需，我们需要存储一些 sessionToken 和一些其他轻量的文件在本地，但是我们无法预测用户最终使用我们 SDK 的运行环境，因此我们无法采用通用的文件存储的解决方案来满足我们的需求。

## 问题的解决

 > 面向接口/协议编程 和 依赖注入

 没错，答案就在我们前面讲过的地方，既然我们无法在一套代码里面实现针对所有平台的文件读写操作，那么解决方案就是我定义一个如何读写文件的接口，然后让运行环境实现这个接口，最后在我需要调用这个接口的地方，用依赖注入的方式将接口的实例传递给调用者，问题自然就解决了。

 ### 定义接口

```swift
public protocol IKVStorage {

    func set(key: String, value: String) -> Observable<String>

    func remove(key: String) -> Observable<Bool>

    func get(key: String) -> Observable<String?>

    func saveJSON(key: String, value: [String: Any]) -> Observable<String>
}
```
```typescript
export interface IStorage {
    add(key: string, value: any): Promise<any>
    remove(key: string):  Promise<boolean>;
    get(key: string): Promise<string>;
}
```
```java
public interface IKVStorage {

    String set(String key, String value);

    boolean remove(String key);

    String get(String key);

    String setJson(String key, Map<String, Object> value);
}

```

### 依赖注入

```swift
public class RxAVUser: RxAVObject {

    init() {
        super.init(className: "_User")
        RxAVUser.kvStorageController = AVCorePlugins.sharedInstance.kvStorageController
    }
        
    static var _kvStorageController: IKVStorage = AVCorePlugins.sharedInstance.kvStorageController
    static var kvStorageController: IKVStorage {
        get {
            return _kvStorageController
        }
        set {
            _kvStorageController = newValue
        }
    }
}
```
```typescript
// 待补充
```
```java
public class RxAVUser extends RxAVObject {
    
    public RxAVUser() {
        super("_User");
        RxAVUser.kvStorage =  RxAVCorePlugins.getInstance().getKVStorage();
    }

    static IKVStorage kvStorage;

    static IKVStorage getKvStorage() {
        if (RxAVUser.kvStorage == null) {
            RxAVUser.kvStorage = RxAVCorePlugins.getInstance().getKVStorage();
        }
        return RxAVUser.kvStorage;
    }
}
```

而针对运行环境的差异化，针对每一个运行环境下单独开发一个小的包，它实现了 SDK 需要注入的接口：


```swift
// 因为 swift 目前主要还是在 iOS 上使用，因此在 SDK 内部默认提供了一个实现，就是使用 UserDefaults.standard 来操作本地缓存
public class RxKVStorage: IKVStorage {

    public func set(key: String, value: String) -> Observable<String> {
        UserDefaults.standard.set(value, forKey: key)
        return Observable.from([value])
    }

    public func get(key: String) -> Observable<String?> {
        let value = UserDefaults.standard.string(forKey: key)
        return Observable.from([value])
    }

    public func remove(key: String) -> Observable<Bool> {
        UserDefaults.standard.removeObject(forKey: key)
        return Observable.from([true])
    }

    public func saveJSON(key: String, value: [String: Any]) -> Observable<String> {
        let jsonString = value.JSONStringify()
        return self.set(key: key, value: jsonString)
    }
}
```
```typescript
import { RxAVClient, IStorage } from 'rx-lean-js-core';

export class BrowserStorage implements IStorage {
    add(key: string, value: any): Promise<any> {
        return new Promise<any>((resolve, reject) => {
            try {
                let dataStr = JSON.stringify(value);
                window.localStorage.setItem(key, dataStr);
                resolve(dataStr);
            } catch (error) {
                reject(error);
            }
        });
    }
    remove(key: string): Promise<boolean> {
        return new Promise<any>((resolve, reject) => {
            try {
                window.localStorage.removeItem(key);
                resolve(key);
            } catch (error) {
                reject(error);
            }
        });
    }
    get(key: string): Promise<string> {
        return new Promise<any>((resolve, reject) => {
            try {
                let dataStr = window.localStorage.getItem(key);
                resolve(dataStr);
            } catch (error) {
                reject(error);
            }
        });
    }
}
```
```java
// 待补充，在 Android 上可以直接使用 File 对象来讲文件存储在 SD 卡或者手机自带的存储
```

然后在运行环境启动时，初始化 SDK，并且将插件注入到 SDK 中：


```swift
// 因为 SDK 默认提供了， 这里 Swift 不需要
```
```typescript
RxAVClient.init({
    log: true,
    plugins: {
        storage: new BrowserStorage(),
    }
});
```
```java
public class RxLeanCloudJavaAndroid {
    public static void link() {
        RxAVCorePlugins.getInstance().setJson(RxLeanCloudJavaAndroidJSONClient.getInstance());
        RxAVCorePlugins.getInstance().setHttpClient(RxLeanCloudJavaAndroidHttpClient.getInstance());
        RxAVCorePlugins.getInstance().setKVStorage(RxLeanCloudJavaAndroidStorage.getInstance());
    }
}

LeanCloud.initialize("uay57kigwe0b6f5n0e1d4z4xhydsml3dor24bzwvzr57wdap","kfgz7jjfsk55r5a8a3y4ttd3je1ko11bkibcikonk32oozww");
LeanCloud.toggleLog(true);
RxLeanCloudJavaAndroid.link();
```