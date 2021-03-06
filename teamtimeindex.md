---
layout: teamtimedefault
---
<link rel="stylesheet" type="text/css" href="{{ '/static/css/article-list.css' | prepend: site.baseurl | prepend: site.url}}" />

<div class="row index">
  <div class="col-sm-10 col-sm-offset-1 col-lg-9 col-lg-offset-1_5">
      <div>
        <section class="category-slice" post-cate="All">
          {% for post in site.posts %}
            <article>
                <header>
                <a href="{{ site.baseurl | prepend: site.url  }}/archive/#{{ post.date | date: '%Y-%m-%d' }}"><span class="octicon octicon-calendar"></span>&nbsp;<span>{{ post.date | date: "%Y-%m-%d" }}</span></a>
                </header>
                <div class="module">
                  <p class="title" style="color:#1e7293;"> {{ post.title }}</p>
                  <p>{% if post.excerpt.size > 32 %}{{ post.excerpt }}{% else %}{{ post.content | strip_html | strip_newlines | truncate: 300 }}{% endif %}</p>
                  <footer>
                      {% for tag in post.tags %}
                      <a class="word-keep" href="{{ site.baseurl | prepend: site.url }}/tags/#{{ tag }}"><span class="octicon octicon-tag"></span>&nbsp;{{ tag }}</a>
                      {% endfor %}
                      <span class="word-keep pull-right">
                          <a class="readmore" href="{{ post.url | prepend: site.baseurl | prepend: site.url }}">Read More</a>
                      </span>
                  </footer>
                </div>
            </article>
          {% endfor %}
        </section>
      </div>
  </div>
</div>