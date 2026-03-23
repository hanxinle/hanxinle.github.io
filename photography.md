---
layout: page
title: 我的摄影
image: assets/images/pic02.jpg
show_tile: true
nav-menu: true
---

<section id="one">
<div class="inner">
<header class="major">
<h1>我的摄影作品</h1>
<p>记录生活中的美好瞬间。</p>
</header>

<!-- 标签筛选 -->
<h2>🏷️ 标签</h2>
<div class="posts" id="tag-filters">
<button class="button small active" data-filter="all">全部</button>
{% assign photo_tags = "" | split: "" %}
{% for post in site.posts %}
  {% if post.categories contains "photography" %}
    {% if post.tags %}
      {% for tag in post.tags %}
        {% unless photo_tags contains tag %}
          {% assign photo_tags = photo_tags | push: tag %}
        {% endunless %}
      {% endfor %}
    {% endif %}
  {% endif %}
{% endfor %}
{% assign photo_tags = photo_tags | sort %}
{% for tag in photo_tags %}
<button class="button small" data-filter="{{ tag | slugify }}">{{ tag }}</button>
{% endfor %}
</div>

<hr class="major" />

<!-- 摄影文章列表 -->
<h2>📷 作品列表</h2>
<div class="posts" id="posts-list">
{% for post in site.posts %}
  {% if post.categories contains "photography" %}
<article data-tags="{% for tag in post.tags %}{{ tag | slugify }} {% endfor %}">
<header>
<h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
<p class="meta">
<time>{{ post.date | date: "%Y-%m-%d" }}</time>
{% if post.tags %} | 标签：
{% for tag in post.tags %}
<span class="tag-badge" data-filter="{{ tag | slugify }}">{{ tag }}</span>
{% endfor %}
{% endif %}
</p>
</header>
</article>
  {% endif %}
{% endfor %}
</div>

</div>
</section>

<style>
.tag-badge { cursor: pointer; color: #2471a3; }
.tag-badge:hover { text-decoration: underline; }
#tag-filters .button { margin: 5px; }
#tag-filters .button.active { background: #2471a3; color: #fff; }
</style>

<script>
document.addEventListener('DOMContentLoaded', function() {
    const filterButtons = document.querySelectorAll('#tag-filters .button');
    const articles = document.querySelectorAll('#posts-list article');
    
    filterButtons.forEach(function(button) {
        button.addEventListener('click', function() {
            const filter = this.getAttribute('data-filter');
            filterButtons.forEach(function(btn) { btn.classList.remove('active'); });
            this.classList.add('active');
            articles.forEach(function(article) {
                const tags = article.getAttribute('data-tags');
                if (filter === 'all' || tags.includes(filter)) {
                    article.style.display = '';
                } else {
                    article.style.display = 'none';
                }
            });
        });
    });
    
    document.querySelectorAll('.tag-badge').forEach(function(tag) {
        tag.addEventListener('click', function() {
            const filter = this.getAttribute('data-filter');
            filterButtons.forEach(function(btn) {
                btn.classList.toggle('active', btn.getAttribute('data-filter') === filter);
            });
            articles.forEach(function(article) {
                const tags = article.getAttribute('data-tags');
                if (tags.includes(filter)) { article.style.display = ''; }
                else { article.style.display = 'none'; }
            });
        });
    });
});
</script>
