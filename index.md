---
title: Home
layout: home
nav_order: 1
description: "Blogs of neural-koala."
permalink: /
---

Blogs of neural-koala.

{% for doc in site.docs %}
## [{{ doc.title }}]({{ doc.url | relative_url }})
{{ doc.content | strip_html | truncatewords: 120 }}
{% endfor %}
