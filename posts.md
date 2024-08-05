---
layout: default
title: "blog"
permalink: /posts/
lang: en
---

<div class="container mt-5">
  <div class="col-md-12">
    <h3 class="fw-bold border-bottom pb-3 mb-5">blog posts</h3>
    {% for post in site.posts %}
      {% if post.lang == page.lang %}
        <div class="col-md-12 mb-4">
          <b><a href="{{ post.url | relative_url }}">{{ post.title }}</a></b> ({{ post.date | date: "%B %-d, %Y" }})
        </div>
      {% endif %}
    {% endfor %}
  </div>
</div>