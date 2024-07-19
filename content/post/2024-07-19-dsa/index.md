---
title: 使用 OpenGL 4.5 DSA
description: 取代状态机
slug: 2024-07-19-dsa
date: 2024-07-19 00:00:00+0000
categories:
    - development
tags:
    - java
    - opengl
---

OpenGL 4.5 中加入了 DSA（Direct State Access），能减少大量状态切换。本文介绍与 DSA 有关的一些常用函数。

本文涉及的函数如果参数没有变化则不标出。

## Program、着色器

Program 和着色器本身已经是 DSA 设计，这里不多介绍。注意把`glUniform*(location, ...)`
替换为`glProgramUniform*(program, location, ...)`即可。

## VAO、缓冲区

创建 VAO（Vertex Array Object）改用`glCreateVertexArrays`。

先前的版本中 VAO 与 VBO（Vertex Buffer Object）无直接关系，OpenGL 通过 VAO 设置的顶点属性读取数据，而顶点属性的`offset`则与状态机绑定的 VBO 有关。

OpenGL 4.3 开始允许 VAO 绑定 VBO。在 DSA 中使用`glVertexArrayVertexBuffer(vao, bindingIndex, vbo, offset, stride)`，其中：

- `bindingIndex`：VBO 的索引
- `offset`：VBO 首个元素的偏移量（字节）
- `stride`：VBO 各个元素的间隔（字节）

使用`glVertexArrayAttribBinding(vao, attributeIndex, bindingIndex)`把顶点属性绑定到指定的`bindingIndex`。

设置顶点属性格式改用`glVertexArrayAttribFormat(vao, attributeIndex, size, type, normalized, relativeOffset)`，`relativeOffset`为顶点属性在`bindingIndex`指定的 VBO 中的偏移量。

启用、禁用顶点属性改用`glEnableVertexArrayAttrib/glDisableVertexArrayAttrib(vao, index)`。

设置 EBO（Element Buffer Object）使用`glVertexArrayElementBuffer(vao, ebo)`。

设置缓冲区的数据改用`glNamedBufferData(buffer, size, data, usage)`、`glNamedBufferSubData(buffer, offset, size, data)`、`glNamedBufferStorage(buffer, size, data, flags)`。

以交错缓冲区（Interleaved buffer）为例（伪代码）：

```java
void setupVertexArray() {
    // 创建对象
    int vao = gl.createVertexArrays();
    int vbo = gl.createBuffers();
    int ebo = gl.createBuffers();
    // 设置数据
    gl.namedBufferData(vbo, new float[]{
        // Position         // Color
         0.0f,  0.5f, 0.0f, 1.0f, 0.0f, 0.0f,
        -0.5f, -0.5f, 0.0f, 0.0f, 1.0f, 0.0f,
         0.5f, -0.5f, 0.0f, 0.0f, 0.0f, 1.0f
    }, GL.STATIC_DRAW);
    gl.namedBufferData(ebo, new int[]{0, 1, 2}, GL.STATIC_DRAW);
    // 设置顶点属性
    gl.vertexArrayVertexBuffer(vao, 0, vbo, 0L, 6 * Float.BYTES);
    gl.vertexArrayElementBuffer(vao, ebo);
    gl.enableVertexArrayAttrib(vao, 0);
    gl.enableVertexArrayAttrib(vao, 1);
    gl.vertexArrayAttribFormat(vao, 0, 3, GL.FLOAT, false, 0);
    gl.vertexArrayAttribFormat(vao, 1, 3, GL.FLOAT, false, 3 * Float.BYTES);
    gl.vertexArrayAttribBinding(vao, 0, 0);
    gl.vertexArrayAttribBinding(vao, 1, 0);
}
```

## 纹理

创建纹理改用`glCreateTextures(target)`。

DSA 不支持直接设置纹理数据，需要先用`glTextureStorage*(texture, levels, internalFormat, ...)`创建存储，再用`glTextureSubImage*(texture, level, ..., format, type, pixels)`设置数据。

设置纹理参数改用`glTextureParameter*(texture, paramName, ...)`。

生成 Mipmap 改用`glGenerateTextureMipmap(texture)`。

激活纹理单元、绑定纹理改用`glBindTextureUnit(unit, texture)`，其中`unit`从数字`0`（而不是`GL_TEXTURE0`）开始。

## 最后

[OpenGL API 索引](https://registry.khronos.org/OpenGL-Refpages/gl4/)
