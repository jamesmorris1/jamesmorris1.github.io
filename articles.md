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
.series-badge {
  display: inline-block;
  background-color: #159957;
  color: white;
  padding: 3px 8px;
  border-radius: 3px;
  font-size: 0.8em;
  margin-left: 10px;
}
</style>

{% for post in site.posts %}
  <div class="article-box">
    <h3>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      {% if post.series %}
        <span class="series-badge">Series: Part {{ post.series_part }}</span>
      {% endif %}
    </h3>
    <p class="article-date">{{ post.date | date: "%B %d, %Y" }}</p>
  </div>
{% endfor %}
