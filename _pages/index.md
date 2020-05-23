---
layout: defaults/page
permalink: index.html
narrow: true
show_tags: true
title: Welcome to Felix's tech notes!
---


### Recent Posts

{% for post in site.posts limit:3 %}
{% include components/post-card.html %}
{% endfor %}


