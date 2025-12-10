---
layout: default
title: Articles
---

# Articles

<style>
.article-box {
  border: 1px solid #ddd;
  padding: 15px;
  margin-bottom: 15px;
  border-radius: 5px;
  background-color: #f9f9f9;
}
.article-box h3 {
  margin-top: 0;
  font-size: 1.1em;
}
.article-date {
  color: #666;
  font-size: 0.9em;
}
</style>

{% for post in site.posts %}
  <div class="article-box">
    <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
    <p class="article-date">{{ post.date | date: "%B %d, %Y" }}</p>
  </div>
{% endfor %}
