---
layout: default
title: "购物攻略"
permalink: /category/shopping-guide/
description: "购物攻略分类页面 - 像素信标为您提供各种购物优惠信息和攻略，包括电商活动、红包雨时间表等。"
published: false
sitemap: false
---

<h1>分类：{{ page.title }}</h1>

<p>这里汇集了各种购物优惠信息和攻略，帮助您省钱购物。</p>

<ul>
{% for post in site.categories['购物攻略'] %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span> - {{ post.date | date: "%Y-%m-%d" }}</span>
    {% if post.description %}
    <p>{{ post.description }}</p>
    {% endif %}
  </li>
{% endfor %}
</ul>