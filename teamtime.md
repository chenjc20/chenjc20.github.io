---
layout: teamtimemain
---
{% for post in site.posts %}
  {% if post.description %}
<article class="post">
  {% if post.img %}
    <a class="post-thumbnail" style="background-image: url({{"/assets/img/" | prepend: site.baseurl | append : post.img}})"></a>
  {% endif %}
  <div class="post-content">
    <h2 class="post-title">
	{% if post.link %}
	<a href="{{post.url | prepend: site.baseurl}}">{{post.title  | markdownify }}</a></h2>
	{% else %}
	<a>{{post.title  | markdownify }}</a></h2>
	{% endif %}
    <p>{{ post.content | strip_html | truncatewords: 15  | markdownify }}</p>
    <span class="post-date">{{post.date | date: '%Y, %b %d'}}&nbsp;&nbsp;&nbsp;—&nbsp;</span>
    <span class="post-words">{{ post.author}}</span>
  </div>
</article>
  {% endif %}
{% endfor %}