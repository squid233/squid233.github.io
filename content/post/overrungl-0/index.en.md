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
  - title: GitHub
    description: OverrunGL is open-source
    website: https://github.com/Over-Run/overrungl
  - title: Wiki
    description: OverrunGL Wiki
    website: https://github.com/Over-Run/overrungl/wiki
  - title: Discussions
    description: Discuss about OverrunGL
    website: https://github.com/Over-Run/overrungl/discussions
  - title: Javadoc
    description: OverrunGL references
    website: https://over-run.github.io/overrungl/
---

Java 22 has been released, and the [FFM API](https://openjdk.org/jeps/454) is finalized.
In this article, I will introduce a library that uses this API called OverrunGL.

## Introduction

OverrunGL is a Java library that allows accessing C libraries.

See the links below for more information.

## Download

You can import OverrunGL with Maven coordinates.
There is a [generator](https://over-run.github.io/overrungl-gen/) for it.

## Function Invocation

OverrunGL defines functions with interfaces and generates implementation at runtime.

Compare with LWJGL 3:

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

The OpenGL module has supported modular loading.
You can load only the functions you need.
See this example:

```java
interface MyFunctions
extends GL10C, GL20C {}

void main() {
    var gl = GLLoader.loadContext(MethodHandles.lookup(), flags, MyFunctions.class);
}
```

This code only loads `GL10C` and `GL20C`.
It will not load other classes that `GL` extends,
so it will improve the bootstrap performance.

{{< admonition tip >}}
An OpenGL state manager which does only invoke GL function
when the state is actually changed.
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

OverrunGL uses `MemorySegment` instead of NIO Buffers.
It is provided by the FFM API.

### Allocation

Use `Arena`, which does only allow allocating but not destroying memory manually.
The memory is only accessible within the scope of the `Arena`.

```java
try (var arena = Arena.ofConfined()) {
    var seg = arena.allocate(ValueLayout.JAVA_INT);
    //...
}
// the memory is automatically released
```

#### Memory Stack

Memory stack (implementation from
[LWJGL 3](https://github.com/LWJGL/lwjgl3/blob/master/modules/lwjgl/core/src/main/java/org/lwjgl/system/MemoryStack.java))
is a kind of `Arena`. It allocates a memory segment once then reuses it.

Use `MemoryStack::stackPush` to push a frame:

```java
try (var stack = MemoryStack.stackPush()) {
    var seg = stack.allocate(ValueLayout.JAVA_INT);
}
```

The push and pop operations must be symmetric.

### Zero-length Segments

Native functions might return a pointer.
If the layout of the returned pointer is not specified,
the FFM API will convert it to a _zero-length_ memory segment.

You can resize it with `MemorySegment::reinterpret`.
This requires native access to the module of the caller.

```text
--enable-native-access=<module-name>
```

### Null Pointer

`NULL` in C is modeled as `MemorySegment.NULL`, which is equivalent to `MemorySegment.ofAddress(0L)`.

## Future Updates

OverrunGL has added GLFW, OpenGL, stb and Native File Dialog.
We plan to add Vulkan, OpenAL, FreeType, Zstandard and Assimp and use Java 25 in the future.
