---
title: "디자인패턴"
layout: archive
permalink: categories/designpattern
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.DesignPattern %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}