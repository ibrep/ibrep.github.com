---
layout: page
title: 博客
permalink: /blog/
order: 1
---
<ul class="post-list">
  {% for article in site.categories.blog %}
     <li>
        <span class="post-meta">{{ article.date | date:"%b %-d, %Y"}}</span>
        <h3><a class="post-link" href="{{ article.url | prepend: site.baseurl }}">{{ article.title }}</a></h3>
     </li>
  {% endfor %}
</ul>

<!---
This is the base Jekyll theme. You can find out more info about customizing your Jekyll theme, as well as basic Jekyll usage documentation at [jekyllrb.com](http://jekyllrb.com/)

You can find the source code for the Jekyll new theme at:
{% include icon-github.html username="jglovier" %} /
[jekyll-new](https://github.com/jglovier/jekyll-new)

You can find the source code for Jekyll at
{% include icon-github.html username="jekyll" %} /
[jekyll](https://github.com/jekyll/jekyll)
--->
