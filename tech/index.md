---
layout: page
title: 技术笔记与总结
excerpt: "Richard's_Blog,是一名php程序员的开发日志,把web开发常见的问题记录总结下来,并分享很多有用的好玩的东西"
hidelogo: true
search_omit: true
---
<br>
<ul class="post-list">
{% for post in site.categories.tech %}
  <li><article><a href="{{ post.url }}">{{ post.title }} <span class="entry-date"><time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %d, %Y" }}</time></span></a></article></li>
      <div class="entry-meta-small">
	  <span>{% if post.tags %}<i class="fa fa-tags"></i>&nbsp;{% endif %}{% for tag in post.tags %}<a href="/tags/#{{ tag }}" title="Posts tagged {{ tag }}">{{ tag }}</a>{% unless forloop.last %}&nbsp;·&nbsp;{% endunless %}{% endfor %}&nbsp;&nbsp;</span>
	  <span>{% capture readtime %}{{ post.content | number_of_words | plus:91 | divided_by:150.0 | append:'.' | split:'.' | first }}{% endcapture %}<i class="fa fa-clock-o"></i>&nbsp;读完需{% if readtime == '0' %} &lt; 1{% else %}{{ readtime }}{% endif %}分钟</span><br>
	  </div>
	  <span class="excerpt">{{ post.excerpt | strip_html | truncatewords:125}}</span>
	  <br><br>
{% endfor %}
</ul>