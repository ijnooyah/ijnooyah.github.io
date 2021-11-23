---
layout: archive
permalink: /categories/java
title: "Post about Java."
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.Java %}

{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}