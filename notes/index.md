---
layout: default
title: Notes
---

# Notes
<input type="text" id="search-input" placeholder="Search...">
<ul id="results-container"></ul>

<ul>
{% for n in site.notes %}
  <li>
    <a href="{{ n.url }}">{{ n.title }}</a>
  </li>
{% endfor %}
</ul>

<script src="https://unpkg.com/simple-jekyll-search/dest/simple-jekyll-search.min.js"></script>

<script>
SimpleJekyllSearch({
  searchInput: document.getElementById('search-input'),
  resultsContainer: document.getElementById('results-container'),
  json: '/search.json',
  searchResultTemplate: '<li><a href="{url}">{title}</a></li>',
  noResultsText: 'No results found'
});
</script>