---
layout: default
title: HOME
---

<p align="center">DERNIERS ARTICLES</p><br>

 {% for post in site.posts %}
  <article>
    <h2>
        {{ post.title }}
    </h2>
    <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_string }}</time><br>
    {{ post.content | strip_html | truncatewords:75}}<br>
            <a href="{{ post.url }}">Read more...</a><br><br>
  </article>
{% endfor %}
