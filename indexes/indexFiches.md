---
layout: default
permalink: /fiches/
title: Fiches
---

 {% for post in site.tags.fiche %}
  <article>
    <h2>
        <a href="{{ post.url }}">{{ post.title }}</a>
    </h2>
    <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_string }}</time><br>
  </article>
{% endfor %}
