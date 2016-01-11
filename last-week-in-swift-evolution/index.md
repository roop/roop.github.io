---
layout: default
permalink: /last-week-in-swift-evolution/index.html
title: "Last week in Swift Evolution: Archive"
rss: /last-week-in-swift-evolution/rss.xml
---

<div class="post-index-container">

  <aside class="roop-intro">
  <p>A weekly summary of selected updates from the Swift Evolution
  repository and mailing list.</p>
  </aside>

  <header class="post-index-header"><h1>Last week in Swift Evolution</h1></header>

  {% for post in site.categories.last-week-in-swift-evolution %}
  <div class="post-index">
    {% include post-summary.html %}
    <hr />
  </div>
  {% endfor %}

  <div style="height: 3em;"></div>

</div>

