---
layout: page
title: Posts
permalink: /posts/
---

<ul>
  {% for post in site.posts %}
    <li>
      <span class="post-meta">{{ site.time | date_to_long_string }}</span>
      <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
      <p>{{ post.excerpt }}</p>
    </li>
  {% endfor %}
</ul>

