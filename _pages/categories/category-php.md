---
layout: archive
permalink: /categories/php
title: "Post about PHP."
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.PHP %}

{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}