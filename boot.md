---
layout: page
title: Boot
permalink: /boot/
order: 4
---
<ul class="post-list">
  {% for article in site.categories.boot %}
     <li>
        <span class="post-meta">{{ article.date | date:"%b %-d, %Y"}}</span>
        <h3><a class="post-link" href="{{ article.url | prepend: site.baseurl }}">{{ article.title }}</a></h3>
     </li>
  {% endfor %}
</ul>
