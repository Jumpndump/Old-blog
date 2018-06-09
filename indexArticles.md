---
layout: default
permalink: /articles/
title: Articles
---
 
 {% for post in site.tags.article %}
  <article>
    <h2>
        {{ post.title }}
    </h2>
    <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_string }}</time><br>
    {{ post.content | strip_html | truncatewords:75}}<br>
            <a href="{{ post.url }}">Read more...</a><br><br>
  </article>
{% endfor %}
