---
layout: page
title: 我的技术文章
image: assets/images/pic11.jpg
show_tile: true
nav-menu: true
---

<section id="one">
<div class="inner">
<header class="major">
<h1>我的技术文章</h1>
<p>共 {{ site.posts | where_exp: "post", "post.categories != 'photography'" | size }} 篇文章，记录学习与工作中的技术心得</p>
</header>

<!-- 标签筛选 -->
<h2 id="tags">🏷️ 标签</h2>
<div class="posts" id="tag-filters">
<button class="button small active" data-filter="all">全部</button>
{% assign tech_tags = "" | split: "" %}
{% for post in site.posts %}
  {% unless post.categories contains "photography" %}
    {% if post.tags %}
      {% for tag in post.tags %}
        {% unless tech_tags contains tag %}
          {% assign tech_tags = tech_tags | push: tag %}
        {% endunless %}
      {% endfor %}
    {% endif %}
  {% endunless %}
{% endfor %}
{% assign tech_tags = tech_tags | sort %}
{% for tag in tech_tags %}
<button class="button small" data-filter="{{ tag | slugify }}">{{ tag }}</button>
{% endfor %}
</div>

<hr class="major" />

<!-- 文章列表 -->
<h2 id="all-posts">📝 所有文章</h2>
<div class="posts" id="posts-list">
{% for post in site.posts %}
  {% unless post.categories contains "photography" %}
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
  {% endunless %}
{% endfor %}
</div>

</div>
</section>

<style>
.tag-badge {
    cursor: pointer;
    color: #2471a3;
}
.tag-badge:hover {
    text-decoration: underline;
}
#tag-filters .button {
    margin: 5px;
}
#tag-filters .button.active {
    background: #2471a3;
    color: #fff;
}
</style>

<script>
document.addEventListener('DOMContentLoaded', function() {
    const filterButtons = document.querySelectorAll('#tag-filters .button');
    const articles = document.querySelectorAll('#posts-list article');
    
    filterButtons.forEach(function(button) {
        button.addEventListener('click', function() {
            const filter = this.getAttribute('data-filter');
            
            // 更新按钮状态
            filterButtons.forEach(function(btn) { btn.classList.remove('active'); });
            this.classList.add('active');
            
            // 筛选文章
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
    
    // 点击标签直接筛选
    document.querySelectorAll('.tag-badge').forEach(function(tag) {
        tag.addEventListener('click', function() {
            const filter = this.getAttribute('data-filter');
            filterButtons.forEach(function(btn) {
                btn.classList.toggle('active', btn.getAttribute('data-filter') === filter);
            });
            articles.forEach(function(article) {
                const tags = article.getAttribute('data-tags');
                if (tags.includes(filter)) {
                    article.style.display = '';
                } else {
                    article.style.display = 'none';
                }
            });
        });
    });
});
</script>
