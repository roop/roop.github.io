---
layout: default
permalink: /index.html
title: "roopc.net"
---

<div class="post-index-container">

  <aside class="roop-intro">
  <p>{% include roop-intro.html %}</p>
  </aside>

  <header class="post-index-header"><h1>Latest blog posts</h1></header>

  {% for post in site.categories.posts limit:3 %}
  <div class="post-index">
    {% include post-summary.html %}
    <hr />
  </div>
  {% endfor %}

</div>

{% include blog-nav.html %}
