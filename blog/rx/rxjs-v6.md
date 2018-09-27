---
date: "2018-09-27"
title: "RxJS 6 的变化"
category: "Coding"
---

<!-- TOC -->

- [有一些惊喜](#%E6%9C%89%E4%B8%80%E4%BA%9B%E6%83%8A%E5%96%9C)
- [pipe 成为了主权公民](#pipe-%E6%88%90%E4%B8%BA%E4%BA%86%E4%B8%BB%E6%9D%83%E5%85%AC%E6%B0%91)
- [from/pipe/of/from/interval/merge/fromEvent](#frompipeoffromintervalmergefromevent)
- [总结](#%E6%80%BB%E7%BB%93)

<!-- /TOC -->
## 有一些惊喜

距离我使用和了解 Rx 已经过去了接近两年，近来重新利用业余时间学习新东西，在改造 Parse 的 SDK 的时候，忽然发现我 npm install 一下，我的所有代码都跑不了了，看了看报错是 Observable 对象的 map 等操作符被提升为一个全局的操作符（其实就是函数），忽然间我恍惚看见了 JS 里面的一等公民函数在 RxJS 里面重新站起来了。具体的变化我们可以逐步分析一下。


## pipe 成为了主权公民

先看看变化，RxJS 5 的代码如下：

```js
source
 .map(x => x + x)
 .mergeMap(n => of(n + 1, n + 2)
 .filter(x => x % 1 == 0)
 .scan((acc, x) => acc + x, 0))
 .catch(err => of('error found'))
 .subscribe(printResult);
```

RxJS 6:

```js
source.pipe(map(x => x + x),
 mergeMap(n => of(n + 1, n + 2).
 pipe(filter(x => x % 1 == 0),scan((acc, x) => acc + x, 0))),
 catchError(err => of('error found'))).subscribe(printResult); 
```

流式编程里面两种风格都有其设计上的理念：

链式表达突出的特色是分层明确，下一层拿上一层返回的结果，约等于网络 OSI 分层的概念。
而 pipe 的概念是，彼此之间像同事或者协作者，我们都在同一级，而如果有新的下一级，新建一个 pipe 就可以明确分层了。

无疑这一次的修改是对面向流式编程的一次递进，乍一看会觉得有点多余，实际上，在使用过程中，一个 pipe 可能对原数据分别作了几个小的操作，而这些操作可能彼此之间有一些关联，甚至顺序都可以在未来被调换，而下一层是另一个 pipe，这个就将 Observable 的灵活性再一次提升了一个维度，着实个不错的想法。不知道其他语言 RxJava 和 RxSwift 如何跟进，Rx .NET 基本除了高玩，国内很少有人会用吧（自黑一下）。


## from/pipe/of/from/interval/merge/fromEvent

这些含糊成为一等公民之后不再受 Observable 的对象绑定，有一个好处就是这些操作符可以在未来拓展出各种可能性，并不一定绑定返回的是一个 `Observable` 对象，站在爱好者的角度去捧一下：之后所有的异步/流式/协程在编程领域都是一家亲，开发者只要 from 一下你得到的就是你想要的。

另外可喜可贺的是终于对 websocket 进行了 rx 的封装，一年前我自己封装的时候，遇到的问题还历历在目，我准备比对一下双方选手实现的方式，然后找个喷点去提 pr（手动微笑）。


```js
import { WebSocketSubject, webSocket } from 'rxjs/websocket';
@Injectable()
export class MessagingService {
  public socket: WebSocketSubject<Message>;
  constructor() {
    this.socket = webSocket(environment.apiUrl);
  }
  public send = (message: Message) => {
    this.socket.next(message);
  }
}
```

在 angular 里面可以上面这么用了，或者这是一种双向影响吧，angular 选择 rx 促进双方一起进步。
顺带提一句：

```js
import { ajax } from 'rxjs/ajax';
```

8102 年了，这个还支持了，夭寿了。


## 总结

其实本质上并没有很大的变化，社区也不存在 kpi 的需求，个人感觉就是函数式的编程思想可能更容易让流式编程爱好者进一步去深化，作为国内开发者来说，仅仅只能局限于爱好，并没有看见国内有对这个东西的大面积使用（传说一些新的 iOS 项目都是在用 RxSwift，这个很赞）。

