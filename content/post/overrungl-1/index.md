---
title: OverrunGL 介绍
description: OverrunGL 介绍
slug: overrungl-1
date: 2024-03-03 00:00:00+0000
categories:
    - Development
tags:
    - java
    - overrungl
draft: true
---

Java 22 已经发布，其中加入了 [FFM API](https://openjdk.org/jeps/454)。
本文介绍使用了 FFM API 的 Java 库——OverrunGL。

## 介绍

OverrunGL 是用 Java 22（截至文章更新）编写的库，支持访问 C 库。

相关链接：

- [GitHub](https://github.com/Over-Run/overrungl)
- [Wiki](https://github.com/Over-Run/overrungl/wiki)
- [讨论](https://github.com/Over-Run/overrungl/discussions)
- [Javadoc](https://over-run.github.io/overrungl/)

## 下载

OverrunGL 支持以 Maven 方式导入。可用[生成器](https://over-run.github.io/overrungl-gen/)生成依赖。网络不佳的环境可能需要 10 分钟加载。

## 函数调用

OverrunGL 使用接口生成函数声明，在运行时生成实例。

```java
import overrungl.glfw.GLFW;
GLFW glfw = GLFW.INSTANCE;
void dispose() {
    glfw.terminate();
}
```
