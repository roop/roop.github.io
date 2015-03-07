---
layout: default
permalink: /index.html
title: "roopc.net"
---

<div class="post-index-container">

  <aside class="roop-intro">
  <p>{% include roop-intro.html %}</p>
  </aside>

  <header class="post-index-header"><h1>Latest posts</h1></header>

  {% for post in site.categories.posts limit:3 %}
  <div class="post-index">
    {% include post-summary.html %}
    <hr />
  </div>
  {% endfor %}

  <div style="height: 3em;"></div>

  <header class="post-index-header"><h1>Popular posts</h1></header>
  {% for post in site.categories.posts %}{% if site.popular-posts contains post.id %}
  <div class="post-index">
    {% include post-summary.html %}
    <hr />
  </div>
  {% endif %}{% endfor %}

</div>

{% include blog-nav.html %}
