---
title: OverrunGL 介绍
slug: overrungl-0
date: 2024-03-03 00:00:00+0000
categories:
    - development
tags:
    - java
    - overrungl
    - game-development
links:
  - title: GitHub
    description: GitHub 存储库
    website: https://github.com/Over-Run/overrungl
  - title: Wiki
    description: OverrunGL Wiki
    website: https://github.com/Over-Run/overrungl/wiki
  - title: Discussions
    description: 讨论 OverrunGL
    website: https://github.com/Over-Run/overrungl/discussions
  - title: Javadoc
    description: OverrunGL Javadoc
    website: https://over-run.github.io/overrungl/
---

OverrunGL 全称为 Overrun Game Library，顾名思义就是由 Overrun Organization 开发的游戏库（实际上和游戏没什么关系）。本文介绍 OverrunGL 相对于 LWJGL 3 的区别和特点。

## 介绍

OverrunGL 使用 Java 23 编写，并利用 Java 22 加入的 [FFM API](https://openjdk.org/jeps/454) 来访问各种 C 库。请参考底部的链接获得详细信息。

## 下载

OverrunGL 支持以 Maven 方式导入。可用[生成器](https://over-run.github.io/overrungl-gen/)生成依赖。

## 函数调用

OverrunGL 使用静态方法调用本机函数。例外是 OpenGL，由实例保存函数的地址，并使用实例方法调用本机函数。

对比 OverrunGL 和 LWJGL 3：

```java
// OverrunGL
import overrungl.opengl.GL;
GL gl;
void render() {
    gl.Clear(...);
}

// LWJGL 3
import static org.lwjgl.opengl.GL11C.*;
void render() {
    glClear(...);
}
```

对于 OpenGL，用实例封装可要求方法强制传递上下文（Context），配合`ScopedValue`在多线程环境中可避开`ThreadLocal`。例如：

```java
static final ScopedValue<GL> OpenGL = ScopedValue.newInstance();
void main() {
    GL gl = ...;
    ScopedValue.where(OpenGL, gl).run(this::render);
}

void render() {
    // 无需传递方法参数
    OpenGL.get().Clear(...);
}
```

## 内存管理

传统的 JNI 库用`long`表示内存地址，并用 NIO Buffer 访问内存。OverrunGL 则使用 FFM API 自带的封装`MemorySegment`。

### 内存分配

`MemorySegment`由`SegmentAllocator`创建。JDK 自带`Arena`分配器，其只允许分配内存，关闭后自动释放所分配的内存。

```java
try (var arena = Arena.ofConfined()) {
    var seg = arena.allocate(ValueLayout.JAVA_INT);
    // 访问内存...
}
// 自动释放内存
```

### 内存堆栈

JDK 自带的 Arena 不支持重复使用内存，在多次分配少量内存时效率相对低下，故需要使用内存堆栈保存并重用分配的内存。

OverrunGL 使用`MemoryStack`实现方法中少量内存的分配。`MemoryStack`的基本用法如下：

```java
try (var stack = MemoryStack.pushLocal()) {
    var seg = stack.allocate(ValueLayout.JAVA_INT);
}
```

### 零长内存段

C 库中的函数可能返回指针，如果没有指定返回指针的类型，FFM API 将使用**零长内存段**（zero-length segment），其只表示一个地址。

开放调用者的权限后，可使用`MemorySegment::reinterpret`设置内存段的大小。

```text
--enable-native-access=<模块名>
```

### 空指针

C 中的`NULL`可用`MemorySegment.NULL`表示，且等效于`MemorySegment.ofAddress(0L)`。

## 未来更新

OverrunGL 已支持 GLFW、OpenGL、stb 以及 Native File Dialog Extended。
我们计划在未来添加 Vulkan、OpenAL、FreeType、Zstandard 和 Assimp，并使用 Java 25。
