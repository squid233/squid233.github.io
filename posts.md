---
layout: page
title: Posts
permalink: /posts/
---

<h3>Comments</h3>
<p>If you didn't see the discussion area, please check your Internet connection, or use <a href="https://github.com/squid233/squid233.github.io/discussions" target="_blank" rel="noopener noreferrer">GitHub discussions</a>.</p>
<ul>
  {% for post in site.posts %}
    <li>
      <span class="post-meta">{{ site.time | date_to_long_string }}</span>
      <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
      <p>{{ post.excerpt }}</p>
    </li>
  {% endfor %}
</ul>

