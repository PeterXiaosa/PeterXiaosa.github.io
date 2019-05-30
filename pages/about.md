---
layout: page
title: About
description: 何时才不是一个渣渣...
keywords: Peter Fu，傅慧成
comments: true
menu: 关于
permalink: /about/
---

我是Xiaosa。

码农渣渣一枚。

深知 纸上得来终觉浅，绝知此事要躬行。

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
