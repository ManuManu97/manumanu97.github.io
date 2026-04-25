---
layout: default
title: Projects
---

# Projects

# Notes
<input type="text" id="searchInput" placeholder="Search...">
<ul id="results-container"></ul>


<ul id="projList">
{% for project in site.projects %}
  <li data-title="{{ project.title | downcase }}">
    <a href="{{ project.url }}">{{ project.title }}</a>
  </li>
{% endfor %}
</ul>

<script src="https://unpkg.com/simple-jekyll-search/dest/simple-jekyll-search.min.js"></script>

<script>
const input = document.getElementById("searchInput");
const items = document.querySelectorAll("#projList li");

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