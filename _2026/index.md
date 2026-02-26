---
layout: page
title: "2026 讲座"
description: >
  Missing Semester（MIT IAP 2026）讲义与视频。
permalink: /2026/
phony: true
excerpt: '' # work around a bug
---

<ul class="double-spaced">
  {% assign lectures = site['2026'] | sort: 'date' %}
  {% for lecture in lectures %}
    {% if lecture.phony != true %}
      <li>
        <strong>{{ lecture.date | date: '%-m/%d' }}</strong>:
        {% if lecture.ready %}
          <a href="{{ lecture.url }}">{{ lecture.title }}</a>
        {% elsif lecture.noclass %}
          {{ lecture.title }} [停课]
        {% else %}
          {{ lecture.title }} [即将更新]
        {% endif %}
        {% if lecture.details %}
          <br>
          （{{ lecture.details }}）
        {% endif %}
      </li>
    {% endif %}
  {% endfor %}
</ul>
