---
layout: teamtimepage
title: Archive
permalink: /archive/
icon: octicon-repo
isNavItem: true
styles: archive.css
---

<section id="cd-timeline" class="cd-container">
{% for post in site.posts  %}
    <div class="cd-timeline-block" id="{{ post.date | date: '%Y-%m-%d' }}">
        <div class="cd-timeline-img cd-picture">
        </div>
        <div class="cd-timeline-content">
            <a href="{{ post.url | prepend: site.baseurl | prepend: site.url }}">
            <h4>{{ post.title }}</h4></a>
            {% for tag in post.tags %}
            <a href="{{ site.baseurl | prepend: site.url }}/tags/#{{ tag }}" style="font-size:15px;color:gray;"><span class="octicon octicon-tag" ></span>&nbsp;{{ tag }}</a>
            {% endfor %}
            <span class="word-keep pull-right">
            <a><span class="octicon octicon-comment"></span>&nbsp;{{post.author}}</a>
            </span>
            <span class="cd-date">{{ post.date | date: '%Y-%m-%d' }}</span>
        </div>
    </div>
{% endfor %}
</section>