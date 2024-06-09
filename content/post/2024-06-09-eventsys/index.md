---
title: 使用 Reactor 取代 Google EventBus
description: 你还在用 Google EventBus？
slug: 2024-06-09-eventsys
date: 2024-06-09 00:00:00+0000
categories:
    - development
tags:
    - java
    - reactor
---

## 什么是 Reactor

{{< quote source="Project Reactor" url="https://projectreactor.io/" >}}
Reactor 是基于 [Reactive Streams](https://github.com/reactive-streams/reactive-streams-jvm) 规范的第四代响应式库，
用于在 JVM 上构建非阻塞应用程序
{{< /quote >}}

只用来发布事件的话比较简单。废话不多说，直接看代码。

## 事件类

创建一个事件类用于保存数据。

```java
record KeyEvent(int key) {}
```

## 发布、订阅

创建 `Sinks.Many` 和 `Flux`：

```java
public static final Sinks.Many<KeyEvent> PUBLISHER = Sinks.many().multicast().onBackpressureBuffer();
public static final Flux<KeyEvent> SUBSCRIBER = PUBLISHER.asFlux();
```

发布事件：

```java
void publish() {
    PUBLISHER.tryEmitNext(new KeyEvent(1));
}
```

订阅事件：

```java
void subscribe() {
    // lambda 方式
    SUBSCRIBER.subscribe(e -> System.out.println(e.key()));
    // 方法引用
    SUBSCRIBER.subscribe(this::onKey);
}

void onKey(KeyEvent e) {
    System.out.println(e.key());
}
```
