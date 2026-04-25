---
layout: default
title: Projects
---

# Projects

<ul>
{% raw %}{% for project in site.projects %}
  <li>
    <a href="{{ project.url }}">{{ project.title }}</a>
  </li>
{% endfor %}{% endraw %}
</ul>