---
layout: default
permalink: /fiches/
title: Fiches
---
 
 {% for post in site.tags.fiche %}
  <article>
    <h2>
        {{ post.title }}
    </h2>
    <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_string }}</time><br>
    {{ post.content | strip_html | truncatewords:75}}<br>
            <a href="{{ post.url }}">Read more...</a><br><br>
  </article>
{% endfor %}
