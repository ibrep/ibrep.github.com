---
layout: page
title: 硬件 
permalink: /hardware/
order: 3
---
<ul class="post-list">
  {% for article in site.categories.hardware %}
     <li>
        <span class="post-meta">{{ article.date | date:"%b %-d, %Y"}}</span>
        <h3><a class="post-link" href="{{ article.url | prepend: site.baseurl }}">{{ article.title }}</a></h3>
     </li>
  {% endfor %}
</ul>
