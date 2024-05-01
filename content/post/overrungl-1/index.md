---
title: OverrunGL 介绍
slug: overrungl-1
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

Java 22 已经发布，其中加入了 [FFM API](https://openjdk.org/jeps/454)。
本文介绍使用了 FFM API 的 Java 库——OverrunGL。

## 介绍

OverrunGL 支持访问 C 库。请参考底部的链接获得详细信息。

## 下载

OverrunGL 支持以 Maven 方式导入。可用[生成器](https://over-run.github.io/overrungl-gen/)生成依赖。{{< mask >}}网络不佳的环境可能需要 10 分钟加载。{{< /mask >}}

## 函数调用

OverrunGL 使用接口声明函数，在运行时生成实例。

与 LWJGL 3 对比：

```java
// OverrunGL
import overrungl.glfw.GLFW;
GLFW glfw = GLFW.INSTANCE;
void dispose() {
    glfw.terminate();
}

// LWJGL 3
import static org.lwjgl.glfw.GLFW.*;
void dispose() {
    glfwTerminate();
}
```

不难看出，LWJGL 3 使用静态方法，而 OverrunGL 使用实例方法。

对于一些库，使用实例方法能避开`ThreadLocal`。例如；

```java
// 省略导入
// OpenGL
void main() {
    GLFW glfw = ...;
    GL gl = load(glfw::getProcAddress);
    drawSomething(gl);
}

void drawSomething(GL gl) {
    // 假设 Mesh 类中有 render 方法
    mesh.render(gl);
}
```

使用`ScopedValue`：

```java
static final ScopedValue<GL> OpenGL = ScopedValue.newInstance();
void main() {
    GL gl = ...;
    ScopedValue.runWhere(OpenGL, gl, this::drawSomething);
}

void drawSomething() {
    // 无需传递参数
    mesh.render();
}
```

### 模块化加载

OpenGL 模块支持模块化加载，即只加载需要的函数。见下列示例：

```java
interface MyFunctions
extends GL10C, GL20C {}

void main() {
    var gl = GLLoader.loadContext(MethodHandles.lookup(), flags, MyFunctions.class);
}
```

这段代码只加载`GL10C`和`GL20C`，不加载（`GL`继承的）其他类，提高初始化性能。

## 内存管理

OverrunGL 使用 FFM API 提供的`MemorySegment`。

### 内存分配

使用`Arena`分配内存。`Arena`只允许分配，不允许手动释放内存。内存只能在`Arena`的作用域内访问。

```java
try (var arena = Arena.ofConfined()) {
    var seg = arena.allocate(ValueLayout.JAVA_INT);
    //...
}
// 自动释放内存
```

#### 内存堆栈

内存堆栈（从
[LWJGL 3](https://github.com/LWJGL/lwjgl3/blob/master/modules/lwjgl/core/src/main/java/org/lwjgl/system/MemoryStack.java)
移植）属于`Arena`。内存堆栈重用分配的内存。

使用`MemoryStack::stackPush`：

```java
try (var stack = MemoryStack.stackPush()) {
    var seg = stack.allocate(ValueLayout.JAVA_INT);
}
```

### 零长内存段

本机函数可能返回指针。如果没有指定返回指针的类型，FFM API 将使用**零长内存段**（zero-length segment）。

开放调用者的权限时，可使用`MemorySegment::reinterpret`设置大小。

```text
--enable-native-access=<module-name>
```

### 空指针

C 中的`NULL`可用`MemorySegment.NULL`表示，且等效于`MemorySegment.ofAddress(0L)`。

## 未来更新

OverrunGL 已经添加了 GLFW、OpenGL、stb和Native File Dialog。
我们计划在未来添加 Vulkan、OpenAL、FreeType、Zstandard 和 Assimp。
