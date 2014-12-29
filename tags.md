---
layout: page
title: Tags
permalink: /tags/
---

{% assign sorted_tags = (site.tags | sort:0) %}
{% for tagitem in sorted_tags %}

<div id="{{ tagitem[0] }}">

  <!-- for create a heading -->
  <h3> {{ tagitem[0] }} </h3>


  <!-- create the list of posts -->
  <ul class="tag-list">

    <!-- iterate through all the posts on the site -->
    {% for post in site.posts %}

      <!-- list only those which contain the current tag -->
      {% if post.tags contains tagitem[0] %}

        <li><a href="{{ post.permalink }}">{{ post.title }} ({{post.date | date: "%b %-d, %Y"}})</a></li>

      {% endif %}
    {% endfor %}
  </ul>

</div>
{% endfor %}