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

OverrunGL uses static methods to invoke native functions, except for OpenGL, which stores addresses of functions with instance and invoke that by instance methods.

Comparing with LWJGL 3:

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

For OpenGL,
instances can be helpful in multi-threading context by avoiding `ThreadLocal`.
For example:

```java
static final ScopedValue<GL> OpenGL = ScopedValue.newInstance();
void main() {
    GL gl = ...;
    ScopedValue.where(OpenGL, gl).run(this::render);
}

void render() {
    // No need to pass an argument
    OpenGL.get().Clear(...);
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
    // access the memory...
}
// the memory is automatically released
```

### Memory Stack

The `Arena` does not support reuse memory,
which caused the performance decreasing when allocating small memory for multiple times.
For this reason, the memory stack is used to store and reuse the (global) allocated memory.

OverrunGL uses `MemoryStack` to implement the allocation of small memory in methods.
The basic usage of `MemoryStack` is shown below:

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

OverrunGL has added GLFW, OpenGL, stb and Native File Dialog Extended.
We are planning to add Vulkan, OpenAL, FreeType, Zstandard and Assimp and use Java 25 in the future.
