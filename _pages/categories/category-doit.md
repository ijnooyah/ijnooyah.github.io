---
layout: archive
permalink: /categories/doit
title: "Post about Doit."
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.Doit %}

{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
