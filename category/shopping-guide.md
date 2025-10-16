---
layout: page
title: "购物攻略"
permalink: /category/shopping-guide/
description: "购物攻略分类页面 - 像素信标为您提供各种购物优惠信息和攻略，包括电商活动、红包雨时间表等。"
---

# 购物攻略

这里汇集了各种购物优惠信息和攻略，帮助您省钱购物。

<div class="posts">
  {% for post in site.categories.购物攻略 %}
    <article class="post">
      <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
      <time datetime="{{ post.date | date_to_xmlschema }}" class="post-date">{{ post.date | date: "%Y年%m月%d日" }}</time>
      <p>{{ post.description }}</p>
    </article>
  {% endfor %}
</div>