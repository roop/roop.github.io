---
layout: default
permalink: /archive/index.html
title: "Blog Archive"
---

<div class="post-index-container">

  <header class="post-index-header"><h1>{{ page.title }}</h1></header>

  {% for post in site.categories.posts %}

      {% unless post.next %}
      <header class="post-index-header"><h2>{{ post.date | date: '%Y' }}</h2></header>
      {% else %}
          {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %}
          {% capture nyear %}{{ post.next.date | date: '%Y' }}{% endcapture %}
          {% if year != nyear %}
          <header class="post-index-header"><h2>{{ post.date | date: '%Y' }}</h2></header>
          {% endif %}
      {% endunless %}

      <div class="post-index">
        {% include post-summary.html %}
        <hr />
      </div>

  {% endfor %}

</div>

{% include blog-nav.html %}
