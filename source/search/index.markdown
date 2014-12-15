---
layout: default
title: Search
footer: true
---

<script src="/javascripts/libs/jquery.min.js"></script>
<script src="/javascripts/lunr.min.js"></script>
<script src="/javascripts/URI.min.js"></script>
<script src="/javascripts/mustache.js"></script>
<script src="/javascripts/search.js"></script>

<div class="container">
    <div class="page-header">
         <h1>Search Results</h1>
    </div>

<script id="search-results-template" type="text/mustache">
<ul>
{% raw %}
{{ #posts }}
    <li><a href="{{ url }}">{{ title }}</a> - {{ date }}</li>
{{ /posts }}
{% endraw %}
</ul>
</script>

<script id="no-results-template" type="text/mustache">
<ul>
{% raw %}
{{ #noposts }}
    <p>No search results found.</p>
{{ /noposts }}
{% endraw %}
</ul>
</script>

<div id="no-results">
</div>

<div id="search-results">
</div>

</div>