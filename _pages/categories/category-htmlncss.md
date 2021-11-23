---
layout: archive
permalink: /categories/html-css
title: "Post about HTML/CSS."
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories['HTML/CSS'] %}

{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}