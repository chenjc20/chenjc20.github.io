---
layout: teamtimepage
title: Tag
permalink: /tags/
icon: octicon-tag
isNavItem: true
styles: [tagCloud.css,tags.css]
scripts: [tagCloud.js,tags.js]
---

<div class="tagCloud">
    {% for tags in site.tags %}
    <a href="#{{ tags.first.tag }}">{{ tags.first.tag }}</a>
    {% endfor %}
</div>

<div>
    <ul class="tag-box inline">
    {% for tags in site.tags %}
        <li><a href="#{{ tags.first.tag }}">{{ tags.first.tag}}
        <span>{{ tags | last | size }}</span></a></li>
    {% endfor %}
    </ul>
</div>

<div>
    {% for tags in site.tags %}
    <h2 id="{{ tags.first.tag }}">{{ tags.first.tag }}</h2>
    <ul>
        {% for posts in tags  %}{% for post in posts %}{% if post.title != null %}
        <li itemscope><span class="entry-date"><time datetime="{{ post.date | date_to_xmlschema }}" itemprop="datePublished">{{ post.date | date_to_string }}</time></span> &raquo; <a href="{{ post.url | prepend: site.baseurl | prepend: site.url }}">{{ post.title }}</a><span> </span><span class="octicon octicon-book">{{ post.categories }}</span></li>
        {% endif %}{% endfor %}{% endfor %}
    </ul>
    {% endfor %}
</div>


