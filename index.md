---
layout: page
title: CorneringEki
tagline: Eklavya Mirani's Corner
---
{% include JB/setup %}

<ul class="posts">
  {% for post in site.posts %}
    <li><h2><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h2><div class="post-list-date"><i>on <span>{{ post.date | date_to_string }}</span></i></div><br />
    </li>
  {% endfor %}
</ul>


