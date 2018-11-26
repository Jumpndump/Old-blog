---
layout: default
title: HOME
---

<p align="center">TOUS LES ARTICLES</p><br>

 {% for post in site.posts %}
  <article>
    <div class="date"><time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_string }}</time></div>

    <h2>
     <a href="{{ post.url }}">{{ post.title }}</a> <code> {{ post.tag }} </code>
    </h2>

    <br>

  </article>
{% endfor %}
