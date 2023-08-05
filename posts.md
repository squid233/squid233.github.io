---
layout: page
title: Posts
permalink: /posts/
---

<ul>
  {% for post in site.posts %}
    <li>
      <span class="post-meta">{{ post.date | date_to_string }}</span>
      <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
      <p>{{ post.excerpt }}</p>
    </li>
  {% endfor %}
</ul>

