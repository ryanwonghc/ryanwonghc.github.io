---
layout: archive
permalink: /projects/
title: "Project Experience"
author_profile: true
#header:
  #image: "/images/Design/niseko.jpg"

---


{% include group-by-array collection=site.posts field="tags" %}

{% for tag in group_names %}
  {% assign posts = group_items[forloop.index0] %}
  <h2 id="{{ tag | slugify }}" class="archive__subtitle">{{ tag }}</h2>
  {% for post in posts %}
    {% include archive-single.html %}
  {% endfor %}
{% endfor %}
