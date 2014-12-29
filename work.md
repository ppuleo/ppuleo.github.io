---
layout: page
title: Work
permalink: /work/
---


<div class="home">

  <div class="posts">
    {% for post in site.categories.['work'] %}
      <div class="post">
        <a href="{{ post.url | prepend: site.baseurl }}" class="post-link">
          <img class="post-media" src="{{ post.image }}" />
          <p class="post-meta">{{ post.date | date: "%B %-d, %Y" }}</p>
          <h3 class="h2 post-title">{{ post.title }}</h3>
          <p class="post-summary">{{ post.summary }}</p>
        </a>
        <ul class="post-categories">
          {% for tag in post.tags %}
          <li><a href="{{ site.baseurl }}/tags#{{ tag | uri_escape }}">{{ tag }}</a></li>
          {% endfor %}
        </ul>
      </div>
    {% endfor %}
  </div>

</div>