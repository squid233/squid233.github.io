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

Java 22 已经发布，其中加入了 [FFM API](https://openjdk.org/jeps/454)。
本文介绍使用了 FFM API 的 Java 库——OverrunGL。

## 介绍

OverrunGL 支持访问 C 库。请参考底部的链接获得详细信息。

## 下载

OverrunGL 支持以 Maven 方式导入。可用[生成器](https://over-run.github.io/overrungl-gen/)生成依赖。{{< mask >}}网络不佳的环境可能需要 10 分钟加载。{{< /mask >}}

## 函数调用

OverrunGL 使用接口声明函数，在运行时生成实例。

对比 OverrunGL 和 LWJGL 3：

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

可见，LWJGL 3 使用静态方法，而 OverrunGL 使用实例方法。

对于部分库，用实例封装可要求方法强制传递该库需要的上下文（Context），在多线程环境中可避开`ThreadLocal`。例如：

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

也可用`ScopedValue`：

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

上面的代码只加载`GL10C`和`GL20C`，不加载（`GL`继承的）其他类，提高初始化性能。

{{< admonition tip >}}
自定义状态机，只在状态改变时调用 OpenGL 函数。
{{< /admonition >}}

```java
abstract class StateManager implements GL10C, GL11C {
    private int textureBinding2D;

    void bindTexture2D(int texture) {
        if (textureBinding2D != texture) {
            textureBinding2D = texture;
            bindTexture(TEXTURE_2D, texture);
        }
    }

    // 可选的getter...
}

void main() {
    var gl = GLLoader.loadContext(MethodHandles.lookup(), flags, StateManager.class);
    render(gl);
}

void render(StateManager gl) {
    gl.bindTexture2D(0);
}
```

## 内存管理

传统的 JNI 库用`long`表示内存地址，并用 NIO Buffer 访问内存。OverrunGL 则使用 FFM API 自带的封装`MemorySegment`。

### 内存分配

`MemorySegment`由`SegmentAllocator`创建。
JDK 自带`Arena`分配器。`Arena`只允许分配内存，关闭`Arena`后自动释放它分配的内存。

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

OverrunGL 已支持 GLFW、OpenGL、stb 以及 Native File Dialog。
我们计划在未来添加 Vulkan、OpenAL、FreeType、Zstandard 和 Assimp，并使用 Java 25。
