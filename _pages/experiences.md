---
layout: archive
permalink: /experiences/
title: "Experiences"
author_profile: true
header:
  image: "/images/bighouse.jpg"
---

{% include group-by-array collection=site.experiences field="tags" %}

{% for tag in group_names %}
  {% assign experiences = group_items[forloop.index0] %}
  <h2 id="{{ tag | slugify }}" class="archive__subtitle">{{ tag }}</h2>
  {% for experience in experiences %}
    {% include archive-single.html %}
  {% endfor %}
{% endfor %}
