---
layout: default
permalink: /articles/
title: Articles
---

 {% for post in site.tags.article %}
  <article>
  <div class="date"><time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_string }}</time></div><br>
    <h2>
        <a href="{{ post.url }}">{{ post.title }}</a>
    </h2>
      </article>
{% endfor %}
