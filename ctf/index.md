---
layout: default
title: CTF
---

# CTF Writeups

<ul>
{% raw %}{% for c in site.ctf %}
  <li>
    <a href="{{ c.url }}">{{ c.title }}</a>
  </li>
{% endfor %}{% endraw %}
</ul>