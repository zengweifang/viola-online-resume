---
layout: archive
title: "Latest Posts in *basic*"
excerpt: "Introduction of how to build a blog like this from scratch"
---

<div class="tiles">
{% for post in site.categories.basic %}
	{% include post-grid.html %}
{% endfor %}
</div><!-- /.tiles -->
