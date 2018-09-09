---
date: "2018-09-09"
title: "Angular 项目源码的文件组织结构讨论"
category: "Angular"
---
<!-- TOC -->

- [问题的来源](#问题的来源)
- [先从官方示例开始](#先从官方示例开始)
- [其他大神的经验](#其他大神的经验)
    - [按照独立模块严格划分](#按照独立模块严格划分)
    - [按照路由和页面进行划分](#按照路由和页面进行划分)
- [总结](#总结)

<!-- /TOC -->

## 问题的来源
最近手痒又想挖新坑，觉得好久没写 Angular 了，就想看看有没有什么新东西，于是用 cli 新建了一个项目，然后我就抑郁了，我已经不知道怎么去组织源码的结构了，任何一个项目都会遇到这个问题：

> 如何设计源代码的文件目录结构？并且一定要满足如下需求：友好，易读，严谨，功能划分明确。

因为程序员多少都有一些强迫症，所以我开始从各种社区和开源的项目里面去探索一种可能存在的最好的方式。

## 先从官方示例开始
点开 [ Style Guide ][1]，我们发现 Google 的官方不只是告诉了一些文件命名/变量命名/缩写/连字符的规范，也一并给出了一个 compliant 的源码文件的规范，如下是来自官网的示例：

```
<project root>
├── src
    ├── app
    │   ├── core
    │   │    ├── core.module.ts
    │   │    ├── exception.service.ts|spec.ts
    │   │    └── user-profile.service.ts|spec.ts
    │   ├── heros
    │   │    ├── hero
    │   │    │    └── hero.component.ts|html|css|spec.ts
    │   │    ├── hero-list
    │   │    │    └── hero-list.component.ts|html|css|spec.ts
    │   │    ├── shared
    │   │    │    ├── hero-button.component.ts|html|css|spec.ts
    │   │    │    ├── hero.model.ts
    │   │    │    └── hero.service.ts|spec.ts
    │   │    ├── heroes.component.ts|html|css|spec.ts
    │   │    ├── heroes.module.ts
    │   │    └── heroes-routing.module.ts
    │   ├── shared
    │   │    ├── shared.module.ts
    │   │    ├── init-caps.pipe.ts|spec.ts
    │   │    ├── text-filter.component.ts|spec.ts
    │   │    └── text-filter.service.ts|spec.ts
    │   ├── villains
    │   │    ├── villain
    │   │    │     └── ...
    │   │    ├── villain-list
    │   │    │     └── ...
    │   │    ├── shared
    │   │    │     └── ...
    │   │    ├── villains.component.ts|html|css|spec.ts
    │   │    ├── villains.module.ts
    │   │    └── villains-routing.module.ts
    │   ├── app.component.ts|html|css|spec.ts
    │   ├── app.module.ts
    │   └── app-routing.module.ts
    ├── main.ts
    ├── index.html
    └── ...
    ├── README.md
    ├── package.json
    ├── node_modules/...
    ├── ...
    └── .gitignore
```

这个结构比较清晰，也很严谨，但是缺少了如下内容：

- Route 路由模块的示例
- Guard 路由保护器的示例
- Services 自定义 Services

为了继续拓展，我们可以从其他开源项目或者大神的博客分享中去学习一下优秀的姿势水平。

## 其他大神的经验

### 按照独立模块严格划分

首先我们看两篇博客：

- [How to define a highly scalable folder structure for your Angular project][2]
- [Angular Folder Structure][3]

这两篇博客中都提到了如下的结构：

```
- app.module.ts
- app.component.ts
- modules
    - module1
        - components
        - pages
            module1.service.ts
            module1.module.ts
            module1.routes.ts
    - module2
        - components
        - pages
            module2.service.ts
            module2.module.ts
            module2.routes.ts
- shared
    - components
    - mocks
    - models
    - ...
```
上述结构的划分理念是：基于相互独立的功能模块（比如登录/注册就是一个独立的模块）。

并且其中一篇还给出了两个示例项目：

- [https://github.com/ngx-rocket/starter-kit/tree/master/src][4]
- [https://github.com/gothinkster/angular-realworld-example-app][5]


基于这个结构，我个人对这种划分保持中立态度，原因如下：

- 根据经验，大部分的项目彼此之间很难完全独立
- 如果每一个 `module` 里面都有 `pages`/`components`/`services`/`routes` 最后文件夹因为太多重名的，还是会一定的阻碍视线


### 按照路由和页面进行划分

另外一篇博客[如何利用angular-cli组织项目结构][6]

则提到了如下结构：

```
src/app
│  app.component.html
│  app.component.scss
│  app.component.spec.ts
│  app.component.ts
│  app.module.ts
├─layout // 通用布局组件
│      layout.module.ts
└─routes
    │  routes.ts // 路由配置文件
    │  routes.module.ts
    ├─trade // 订单
    │  │  trade.module.ts
    │  ├─list // 订单列表组件目录
    │  └─view // 订单明细组件目录
    └─user // 会员
        │  user.module.ts
        ├─list
        └─view

作者：cipchk
链接：https://www.jianshu.com/p/a11927abab25
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
```

但是仔细分析，这三篇博客提到的结构本质上无差别，在一个路由就等于一个模块的习惯下，这个确实是 Web 前端项目的天然的一种趋势。

而仔细分析，还是有一些差别，后面这一种可能更面向较为简单的项目，例如整个站点上就只有一个核心功能模块，例如电影评分/外卖点餐之类的，一个路由就是一个页面，一个页面就是一个模块，而前一种则可能面向的是较为复杂的混合站点，一些 Admin/SA 的操作也可能包含在路由当中，所以用模块名字来提升到第一梯队，进而用 `pages` 来划分具体的页面。

所以前一种可以说是包含了后一种的理念，而后一种对于单模块应用来说，还是可以使用的。


## 总结

结合官方的示例，加上其他大神的分享，我们不难看出，一个混合多模块多权限多路由多页面的大型复杂的企业级应用，在源码的文件目录上是需要下一些功夫的，一定要做到针对具体的项目具体分析，官网的示例在我看来就是一个纯前端单模块多路由多页面的应用，企业级的应用可以参考[按照独立模块严格划分](#按照独立模块严格划分)里面的介绍今早的划分模块，而在模块里面的页面/路由可以参照官网和[按照路由和页面进行划分](#按照路由和页面进行划分)继续划分，这样可能更科学。


[1]:	https://angular.io/guide/styleguide "Style Guide"
[2]:    https://itnext.io/choosing-a-highly-scalable-folder-structure-in-angular-d987de65ec7
[3]:    https://medium.com/@motcowley/angular-folder-structure-d1809be95542
[4]:    https://github.com/ngx-rocket/starter-kit/tree/master/src
[5]:    https://github.com/gothinkster/angular-realworld-example-app
[6]:    https://www.jianshu.com/p/a11927abab25