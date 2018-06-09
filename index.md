---
layout: default
title: HOME
---

 {% for post in site.posts %}
  <article>
    <h2>
        {{ post.title }}
    </h2>
    {{ post.date | date: "%d-%m-%Y }}<br>
    {{ post.content | strip_html | truncatewords:75}}<br>
            <a href="{{ post.url }}">Read more...</a><br><br>
  </article>
{% endfor %}

## test 1

blabla
