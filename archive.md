---
layout: page
title: All posts
permalink: /archive/
---

<ul class="post-list">
{%- for post in site.posts -%}
  <li>
    <span class="post-meta">{{ post.date | date: site.date_format }}</span> &mdash;
    <a href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
  </li>
{%- endfor -%}
</ul>
