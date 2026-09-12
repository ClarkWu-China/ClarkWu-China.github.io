---
layout: page
permalink: /blog/
title: Blog
nav: true
nav_order: 2
description:
---

{% assign real_posts = site.posts | where: "real_post", true %}

<ul class="post-list">
  {% for post in real_posts %}
    <li>
      <h3><a class="post-title" href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
      <p>{{ post.description }}</p>
      <p class="post-meta">{{ post.date | date: '%B %d, %Y' }}</p>
    </li>
  {% endfor %}
</ul>
