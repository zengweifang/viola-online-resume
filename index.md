---
layout: archive
title: "Latest Posts"
image: 
    feature: cover.jpg
    credit: Hubber's high resolution photo
    creditlink: http://www.universetoday.com/wp-content/uploads/2010/02/The-Majestic-Sombrero-Galaxy-M104.jpg
---

<div class="tiles">
{% for post in site.categories.basic %}
	{% include post-grid.html %}
{% endfor %}
</div><!-- /.tiles -->
