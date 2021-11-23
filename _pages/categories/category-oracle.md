---
layout: archive
permalink: /categories/oracle
title: "Post about Oracle."
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.Oracle %}

{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}