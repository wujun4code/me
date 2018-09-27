---
date: "2018-09-26"
title: "F5 启动 Debug TypeScript，没错，像 C# 那样"
category: "Tools"
---

<!-- TOC -->

- [VSCode NodeJS 的 Debug 支持](#vscode-nodejs-%E7%9A%84-debug-%E6%94%AF%E6%8C%81)
- [基础运行逻辑要清楚](#%E5%9F%BA%E7%A1%80%E8%BF%90%E8%A1%8C%E9%80%BB%E8%BE%91%E8%A6%81%E6%B8%85%E6%A5%9A)
- [开始动手](#%E5%BC%80%E5%A7%8B%E5%8A%A8%E6%89%8B)
    - [1. 确定使用的框架](#1-%E7%A1%AE%E5%AE%9A%E4%BD%BF%E7%94%A8%E7%9A%84%E6%A1%86%E6%9E%B6)
    - [2.划分 Bash/Shell/Task 分别负责的区域](#2%E5%88%92%E5%88%86-bashshelltask-%E5%88%86%E5%88%AB%E8%B4%9F%E8%B4%A3%E7%9A%84%E5%8C%BA%E5%9F%9F)
    - [3.理清需求](#3%E7%90%86%E6%B8%85%E9%9C%80%E6%B1%82)
    - [4.额外的配置项](#4%E9%A2%9D%E5%A4%96%E7%9A%84%E9%85%8D%E7%BD%AE%E9%A1%B9)
- [示例代码](#%E7%A4%BA%E4%BE%8B%E4%BB%A3%E7%A0%81)
- [总结](#%E6%80%BB%E7%BB%93)

<!-- /TOC -->

## VSCode NodeJS 的 Debug 支持

VSCode 有自带的 NodeJS 的 Debug 支持，并且可以很好的支持断点跟踪到 js 源码，而且可以想 Visual Studio Debug C# 代码一样，实时的查看内存中每一个对象的状态，但是对于 TypeScript 要配起来这套环境就有一些难度。

## 基础运行逻辑要清楚

1. TypeScript 的项目到底是运行在浏览器上还是 Node 里面对于 VSCode 来说是需要两个插件来支持的
2. TypeScript 会被编译成 JS 运行，不管是 Node Debug 还是 Chrome Debug，在没有配置 sourceMap 的时候 TypeScript 编译出来的 JS 代码根本无法友好的跟踪断点
3. 单元测试启动的时候，也要做一些 preBuild 的事情，并且单元测试的启动路径和运行环境得搭配 Node 的进程


## 开始动手

先看看我们需要的效果：

`video: https://www.youtube.com/embed/AucP_TOoFyA`

### 1. 确定使用的框架

首先分析了一下各种能在 Node 环境下运行的 JS 单元测试的框架：

1. Mocha
2. Jasmine
3. AVA
4. Jest

比较新的是 AVA，暂时社区没有对它在 VSCode 做插件支持，而 Jasmine 是 Angular 钦定的测试框架，Jest 是 Facebook 钦定的，所以我都不用（就是这么任性）

最后选了 Mocha，原因很简单，是因为 VSCode 内置的 NodeJS 插件集成了  Mocha 的 Debug 脚本……

### 2.划分 Bash/Shell/Task 分别负责的区域

首先编译这一步应该是一个 PreTask，真正调用 Mocha 来 Run TestCases 的时候才应该交给 VSCode 的 Node Debug，
因此在 `.vscode/tasks.json` 编写如下内容：


```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "label": "typescript",
    "tasks": [
        {
            "label": "typescript",
            "type": "shell",
            "command": "tsc",
            "problemMatcher": "$tsc",
            "args": [
                "-p",
                "\"${workspaceFolder}/tsconfig.json\""
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        },
        {
            "label": "build dev",
            "type": "shell",
            "command":"tsc"
        },
        {
            "label": "gulp dev",
            "type": "shell",
            "command":"gulp devCopy"
        },
        {
            "label": "pretest",
            "dependsOn": ["build dev", "gulp dev"]
        }
    ]
}
```

这里只是定义，真正用的是在 `.vscode/launch.json` 里面编写如下内容：


```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [{
            "type": "node",
            "request": "launch",
            "name": "Mocha Tests",
            "preLaunchTask": "pretest",
            "runtimeExecutable": "/usr/local/bin/node",
            "cwd": "${workspaceRoot}",
            "program": "${workspaceFolder}/node_modules/mocha/bin/_mocha",
            "args": [
                "-u",
                "tdd",
                "--timeout",
                "999999",
                "--colors",
                "--recursive",
                "${workspaceRoot}/.bin/test"
            ],
            "runtimeArgs": [
                "--nolazy"
            ],
            "sourceMaps": true,
            "internalConsoleOptions": "openOnSessionStart",
            "outFiles": ["${workspaceRoot}/.bin"],
            // Prevents debugger from stepping into this code :)
            "skipFiles": [
                "node_modules/**/*.js",
                "<node_internals>/**/*.js"
            ]
        }, {
            "name": "Debug Mocha Test Current File",
            "type": "node",
            "request": "launch",
            "preLaunchTask": "pretest",
            "runtimeExecutable": "/usr/local/bin/node",
            "program": "${workspaceFolder}/node_modules/mocha/bin/_mocha",
            "args": [
                "-u",
                "tdd",
                "--timeout",
                "999999",
                "--colors",
                "${workspaceRoot}/.bin/test/**/*${fileBasenameNoExtension}*.js",
            ],
            "runtimeArgs": [
                "--nolazy"
            ],
            "sourceMaps": true,
            "internalConsoleOptions": "openOnSessionStart",
            "outFiles": ["${workspaceRoot}/.bin"],
            "skipFiles": [
                "node_modules/**/*.js",
                "<node_internals>/**/*.js"
            ]
        }
    ]
}
```

上述在 F5 按下的时候实际上它根据每一个启动的 Debug 任务的配置先去执行 `preLaunchTask`，而它的内容来自于 `.vscode/tasks.json`。


### 3.理清需求

第一个叫做 `Mocha Tests`，它会跑遍所有的编译编译目录下的所有 `*.js` 的单元测试文件，而第二个 `Debug Mocha Test Current File`，则只会 Debug 你当前打开的那个 TypeScript 文件，强大吧？



### 4.额外的配置项

需要使用 Gulp/Grunt 搭配 tsc 预编译，另外 `tsconfig.json` 也需要特殊配置一下，详细请参考[实例代码](#示例代码)

## 示例代码

- [RxParse/Parse-SDK-ts](https://github.com/RxParse/Parse-SDK-ts)

## 总结

VSCode 因为彻底的开源和开放，它迭代的功能比 VS 要快，并且因为大部分都针对时下流行语言/流行框架的功能，搭配一些配置的知识可以快速的提高变成效率。




