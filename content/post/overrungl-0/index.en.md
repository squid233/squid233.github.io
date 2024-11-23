---
title: OverrunGL Introduction
slug: overrungl-0
date: 2024-04-12 00:00:00+0000
categories:
    - development
tags:
    - java
    - overrungl
    - game-development
links:
    -   title: GitHub
        description: OverrunGL is open-source
        website: https://github.com/Over-Run/overrungl
    -   title: Wiki
        description: OverrunGL Wiki
        website: https://github.com/Over-Run/overrungl/wiki
    -   title: Discussions
        description: Discuss about OverrunGL
        website: https://github.com/Over-Run/overrungl/discussions
    -   title: Javadoc
        description: OverrunGL references
        website: https://over-run.github.io/overrungl/
---

OverrunGL, whose full name is Overrun Game Library, is developed by Overrun Organization.
This article introduced the differences between OverrunGL and LWJGL 3 as well as the characteristics of OverrunGL.

## Introduction

OverrunGL is written in Java 23, using the [FFM API](https://openjdk.org/jeps/454) added in Java 22, which allows one to
access various C libraries.

See the links below for more information.

## Download

You can import OverrunGL with Maven coordinates.
There is a [generator](https://over-run.github.io/overrungl-gen/) for it.

## Function Invocation

OverrunGL defines functions with interfaces and generates implementation at runtime.

Comparing with LWJGL 3:

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

You can see that LWJGL 3 uses static methods,
but OverrunGL uses instance methods.

For several libraries,
instances might be helpful in multithreading context by avoiding `ThreadLocal`.
For example:

```java
// Imports are omitted
// OpenGL
void main() {
    GLFW glfw = ...;
    GL gl = load(glfw::getProcAddress);
    drawSomething(gl);
}

void drawSomething(GL gl) {
    // Assumes there is a Mesh class with render method
    mesh.render(gl);
}
```

or `ScopedValue`:

```java
static final ScopedValue<GL> OpenGL = ScopedValue.newInstance();
void main() {
    GL gl = ...;
    ScopedValue.runWhere(OpenGL, gl, this::drawSomething);
}

void drawSomething() {
    // No need to pass an argument
    mesh.render();
}
```

### Modular Loading

The OpenGL module supports modular loading i.e. only load the functions that one would need.
See this example:

```java
interface MyFunctions
extends GL10C, GL20C {}

void main() {
    var gl = GLLoader.loadContext(MethodHandles.lookup(), flags, MyFunctions.class);
}
```

This code only loads `GL10C` and `GL20C`.
Besides that, no other methods whose owner classes extended by `GL` will be loaded,
so that the bootstrap performance will be improved.

{{< admonition tip >}}
An OpenGL state manager which does only invoke GL function
when the state is actually changed.
{{< /admonition >}}

```java
abstract class StateManager implements GL10C, GL11C {
    private int textureBinding2D;

    // Add annotation Skip to avoid identifying as an OpenGL function
    @overrun.marshal.gen.Skip
    void bindTexture2D(int texture) {
        if (textureBinding2D != texture) {
            textureBinding2D = texture;
            bindTexture(TEXTURE_2D, texture);
        }
    }

    // an optional getter here...
}

void main() {
    var gl = GLLoader.loadContext(MethodHandles.lookup(), flags, StateManager.class);
    render(gl);
}

void render(StateManager gl) {
    gl.bindTexture2D(0);
}
```

## Memory Management

Traditional JNI libraries use `long` to represent memory address and NIO buffers to access memory.
In OverrunGL, however, it uses `MemorySegment`, which is provided by the FFM API.

### Allocation

The `MemorySegment` is created by `SegmentAllocator`.
A common-used type of segment allocator included in JDK is `Arena`, which only allows allocating
but not destroying memory manually.
The memory is only accessible within the scope of the `Arena`.

```java
try (var arena = Arena.ofConfined()) {
    var seg = arena.allocate(ValueLayout.JAVA_INT);
    //...
}
// the memory is automatically released
```

### Memory Stack

The `Arena` does not support reuse memory,
which caused the performance decreasing when allocating small memory for multiple times.
For this reason, the memory stack is used to store and reuse the (global) allocated memory.

OverrunGL uses [memstack](https://github.com/Over-Run/memstack) to implement the allocation of small memory in methods.
The basic usage of memstack is shown below:

```java
try (var stack = MemoryStack.stackPush()) {
    var seg = stack.allocate(ValueLayout.JAVA_INT);
}
```

The push and pop operations must be symmetric.

### Zero-length Segments

Native functions might return a pointer.
If the layout of the returned pointer is not specified, the FFM API will convert it to a _zero-length_ memory segment,
which only represents a memory address.

After enabling the native access to the caller, you can resize the returned memory segment
with `MemorySegment::reinterpret`.

```text
--enable-native-access=<module-name>
```

### Null Pointer

`NULL` in C is modeled as `MemorySegment.NULL`, which is equivalent to `MemorySegment.ofAddress(0L)`.

## Future Updates

OverrunGL has added GLFW, OpenGL, stb as well as (Extended) Native File Dialog.
We are planning to add Vulkan, OpenAL, FreeType, Zstandard and Assimp and use Java 25 in the future.
