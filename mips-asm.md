---
layout: page
title: 龙芯汇编
permalink: /mips-asm/
order: 1
---
<!--
<ul class="post-list">
	  {% for article in site.categories.mips-asm %}
		<li>
      		<span class="post-meta">{{ article.date | date:"%b %-d, %Y"}}</span>
		<h3><a class="post-link" href="{{ article.url | prepend: site.baseurl }}">{{ article.title }}</a></h3>
		</li>
	  {% endfor %}
</ul>
-->
<ul class="post-list">
	{% assign sorted_articles = site.categories.mips-asm | sort:"order" %}
    {% for article in sorted_articles %}
      <li>
		  <span class="post-meta">{{ article.date | date:"%b %-d, %Y"}}</span>
          <h3><a class="page-link" href="{{ article.url | prepend: site.baseurl }}">{{ article.title }}</a></h3>
      </li>
    {% endfor %}
</ul>
