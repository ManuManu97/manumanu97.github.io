---
layout: default
title: Notes
---

# Notes

<ul>
{% for n in site.notes %}
  <li>
    <a href="{{ n.url }}">{{ n.title }}</a>
  </li>
{% endfor %}
</ul>