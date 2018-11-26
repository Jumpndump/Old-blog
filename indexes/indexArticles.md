---
layout: default
permalink: /articles/
title: Articles
---
<p align="center">ARTICLES</p>
<p align="center">Cette section centralise tous les articles sur divers sujets.</p><br>


 {% for post in site.tags.article %}
  <article>
  <div class="date"><time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_string }}</time></div>
    <h2>
        <a href="{{ post.url }}">{{ post.title }}</a>
    </h2>
      </article>
{% endfor %}
