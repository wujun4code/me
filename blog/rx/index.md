---
date: "2018-02-09"
title: "初探 Rx"
category: "Coding"
---

<!-- TOC -->

- [代码怎么变丑的](#代码怎么变丑的)
- [Rx 不止是语法糖](#rx-不止是语法糖)
- [Rx 好玩在哪里？](#rx-好玩在哪里)
- [Rx 适用的场景](#rx-适用的场景)
- [推荐阅读](#推荐阅读)

<!-- /TOC -->

每个语言都有自己原生独特的异步解决方案，JavaScript 有 Promise，Java 有 FutureTask，Objective-C 有 Block 等等，它们的设计初衷都是为了解决不在主线程做一些耗时较多的操作。因为大家的目的是一样的，因此总有人喜欢搞大新闻想着可以有一套通用的接口和「编码习惯」让这些流行的语言在处理异步的代码看上去长得一样，Rx 的理念就这样诞生了。

## 代码怎么变丑的

首先我们基于 LeanCloud 的 JavaScript SDK 实现如下需求：

> 根据输入的关键字搜索 Todo，找出所有 content 字段包含这个关键字的 Todo 对象

首先我们采用常规的编码模式来实现需求：


```
<script>
  var keywordTextBox = document.querySelector('#keyword');
  keywordTextBox.addEventListener('keyup', (e) => {
    var searchText = e.target.value;
    var query = new AV.Query('Todo');
    query.contains('content', searchText);
    query.find().then(data =>{
      render(data);// 此处为渲染 UI 的代码不做深入探究
    });
  });
</script>
```

但是上述代码有两个比较难过的问题：

1. 当想搜索“写周报”时，输入框可能会存在三种情况，“写”、“写周”、“写周报”。而这三种情况将会发起 3 次请求，前面 2 次是多余的请求。
2. 一开始搜了“写周报”，然后马上改搜索“周会总结”。结果后台返回了“写周报”的搜索结果，执行渲染逻辑后结果框展示了“写周报”的查询结果，而不是当前正在搜索的“周会总结”。

因此我们需要加上定时器以及关键字的过滤来降低请求的频率并且核对查询结果的关键字是否就是最近一次输入的关键字，因此最终的代码大概长这样：

```
<script>
  var keywordTextBox = document.querySelector('#keyword'),
    timer = null,
    currentSearch = '';
  keywordTextBox.addEventListener('keyup', (e) => {
    clearTimeout(timer);
    timer = setTimeout(() => {
      // 声明一个当前搜索的关键字
      currentSearch = '做周报';
      var searchText = e.target.value;
      var query = new AV.Query('Todo');
      query.contains('content', searchText);
      query.find().then(data => {
        if (data.length > 0) {
          var first = data[0];
          var content = first.get('content');
          if (content.indexOf(currentSearch) > -1) {
            render(data);// 此处为渲染 UI 的代码不做深入探究
          }
        }
      });
    },250);// 250 ms 之后实行第二次搜索，避免多余请求
  });
</script>
```

（恕在下直言，这段代码真的好丑

如果改成 RxJS 方式，代码如何呢？

```
<script>
  var inputStream = Rx.Observable.fromEvent(keywordTextBox, 'keyup')
    .debounceTime(250)//每隔 250 ms 才会启动一次监听 keyup 的事件变化
    .pluck('target', 'value')
    .switchMap(searchText => {
      var query = new AV.Query('Todo');
      query.contains('content', searchText);
      var result = query.find();
      return Rx.Observable.fromPromise(result);// 将 Promise 执行的结果转换成一个可订阅的 Observable 对象 
    }).subscribe(data => render(data));
</script>
```

小结：看上去代码好像优雅了一点，可这仅仅是 Rx 的一小部分

## Rx 不止是语法糖

ReactiveX 是Reactive Extensions的缩写，Rx是一个编程模型，目标是提供一致的编程接口，帮助开发者更方便的处理异步数据流。

Rx 的第一眼感觉可能就是一个语法糖，并不能吸引到很多开发者去使用，但是我们从更多场景去了解 Rx 风格的代码带来的好处，也许就不会让人误解它就是个语法糖了。

首先，我们来看一下 Rx 里面的 filter | map | flatMap 等操作符：

```
// 将 AV.Query 查询的结果包装成一个 Observable<Array<AV.Object>>
function queryTodo() {
    var query = new AV.Query('Todo');
    return Rx.Observable.fromPromise(query.find());
}
// 获取所有 content 包含了数字 2 的 Todo 
// 并且为这些 Todo 的 content 数字 2 加上 [] 包装
queryTodo()
    .flatMap(list => Rx.Observable.from(list))// 将 AV.Object 集合拆解成单个 AV.Object 发送给订阅方
    .map(todoObj => todoObj.get('content')) // 获取每一个 Todo 对象的 content
    .filter(content => content.indexOf('2') > -1) // 获取 content 包含 2 的 Todo
    .map(mataContent => mataContent.replace('2', '[2]'))// 将 2 替换成 [2]
    .subscribe(result => {
        console.log(result);
    }, error => {
        
    }, () => {
        console.log('completed.')
    });
```

以上代码用链式语法实现的每一步都阐述了函数的核心需求，将数组对象的拆解，属性获取，条件判断，UI 展现拆开成各自独立精简的 lambda 表达式，串联起来，让客户端的代码变得格外的「顺眼」。


## Rx 好玩在哪里？

第一个实例我们将 DOM 的 addEventListener 包装成了一个 Observable 对象并且也将一个 AV.Query.find 返回的 Promise 包装成了一个 Observable 实现了关键字搜索的功能。
第二个实例我们看到了 Observable 对象是可以进行的各种转换，逻辑判断，属性获取的操作。

我们假设客户端是消费者，数据源（可以是来自服务端也可能是来自内存已有的数据）是生产者

消费者可以只使用 Observable 来替换 addEventListener 和 Promise 发起异步或者同步的请求，加上 Observable 自身携带了操作符（Operators）来消费一个自定义的「产品」

这种编程思想的转换才是 Rx 真正想表达的，这也是 Rx 真正可以在诸多主流语言里面依然活跃的根本原因。

Rx 针对如下的概念进行了统一的封装：

逻辑概念|Rx概念
--|--
数据源（可以是数据库，服务端 REST API，内存里的数据）|可订阅对象 Observable
获取方式（同步，异步，事件回调）| 订阅操作 Subscribe
条件过滤 | filter 操作符
对象转换 | map 操作符
数组降维 | flatMap 操作符

## Rx 适用的场景

- Rx 最显著的特性是使用可观察集合(Observable Collection)来达到集成异步(composing asynchronous)和基于事件(event-based)的编程的效果

例如，在聊天应用中是 Rx 绝对是最佳实践，因为聊天由主动发送和被动通知组成，正好使用 Rx 可以让主动发送和被动通知的代码看上去是相似的，而接收方订阅消息的通知。其伪代码看上去是这样的：

```
// 向 id 为 123 的对话发送一条 hello
LeanCloud.Realtime.send('hello','123').subscribe(sent =>console.log(sent.id));

// 接收 对话 id 为 123 的消息
LeanCloud.Realtime.onMessage.filter(message => message.id == '123').subscribe(received =>console.log(received.content));
```

- Rx 编程允许我们用更合适的方式来构建应用，从而更适合异步操作进行。我们需要接纳数据源的异步特性，而不是试图去管理所有的状态。

例如，工单系统中，用户回复了一个工单，客户端需要将工单的最新状态以及用户回复的内容推送到工程师的所在的客户端，假设每一个工单都拥有一个可订阅的状态变化，那么我们可以让修改工单状态的代码和真正发送推送通知的代码隔离，让每一段代码都处理自己核心的业务逻辑，本质上是为了让各个模块（各自独立的业务函数）之间实现一个基于数据流（而不是基于事件通知）的通信

关于[Rx] 我利用业余时间自己维护了一个基于 Rx 风格的 LeanCloud 客户端 SDK，目前已经有 Swift 和 TypeScript 两个轻度可用和 Java 的开发板，详情请点击 [https://github.com/RxLeanCloud](https://github.com/RxLeanCloud)，它对应的文档在 [http://rxcoding.org/](http://rxcoding.org/)


## 推荐阅读

> [不要把 Rx 用成 Promise](https://zhuanlan.zhihu.com/p/20531896)
