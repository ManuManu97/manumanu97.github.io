---
layout: default
title: CTF
---

# CTF Writeups

<input type="text" id="searchInput" placeholder="Search...">
<ul id="results-container"></ul>


<ul id="ctfList">
{% for c in site.ctf %}
  <li data-title="{{ c.title | downcase }}">
    <a href="{{ c.url }}">{{ c.title }}</a>
  </li>
{% endfor %}
</ul>

<script src="https://unpkg.com/simple-jekyll-search/dest/simple-jekyll-search.min.js"></script>

<script>
const input = document.getElementById("searchInput");
const items = document.querySelectorAll("#ctfList li");

input.addEventListener("keyup", function () {
  const filter = input.value.toLowerCase();

  items.forEach(item => {
    const title = item.getAttribute("data-title");

    if (title.includes(filter)) {
      item.style.display = "";
    } else {
      item.style.display = "none";
    }
  });
});

let visibleCount = 0;

items.forEach(item => {
  const title = item.getAttribute("data-title");

  if (title.includes(filter)) {
    item.style.display = "";
    visibleCount++;
  } else {
    item.style.display = "none";
  }
});

document.getElementById("noResults").style.display =
  visibleCount === 0 ? "block" : "none";
</script>