---
layout: post
title: "Make a Game #2: Shader & Mesh"
author: squid233
comments: false
---

[All posts]({% link series-make-a-game.md %}) · [Repository](https://github.com/squid233/make-a-game)

If you have any questions, please use [discussions](https://github.com/squid233/squid233.github.io/discussions).

---

Since we have a simple window, we can now develop our game.
However, before we can render anything, we can't debug the logic of our game easily,
so we need to write a renderer.

## Meshes

We need meshes to describe the vertex layout and data of our game objects.

Let's first implement the _vertex layout_.
A vertex layout describes what attributes it has, and where they are.

### Vertex Attribute

A _vertex attribute_ indicates the properties of a program attribute.

- `index`: The index of the attribute. We can use it to specify the offset of the vertex data in a buffer object.
- `size`: Indicates how many elements are in it.
- `type`: The type of each element.
- `normalized`: Specifies that whether the value of each element should be normalized when passing the values to the shader.

We will add `POSITION` and `COLOR` attribute at now, and more in the future.
We will also create a new package called `render`, as we will put everything about rendering in it.

{% highlight java %}
public enum VertexAttribute {
    POSITION(0, 3, GL.FLOAT, false),
    COLOR(1, 4, GL.UNSIGNED_BYTE, true);
    //definition...
}
{% endhighlight %}

### Vertex Layout

We use the vertex layout to store the attributes and offsets.

## Shaders

## Game Renderer

## Modular

---
