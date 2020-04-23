---
title: "Hi, I'm **Ryan**"
layout: splash
permalink: /
date: 2020-04-20T11:48:41-04:00
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /images/Design/tahoe.jpg
  actions:
    - img: "/images/Design/github.jpg"
      url: "https://ryanwonghc.github.io/about/"
    - img: "/images/Design/linkedin.jpg"
      url: "https://ryanwonghc.github.io/projects/"
  #caption: "Photo credit: [**wallpaperaccess**](https://wallpaperaccess.com/lake-tahoe)"
excerpt: |
    Welcome to my site!
    ---
    <br>
    <br>
    <br>
    <br>
    <br>
    <br>
---

{% if page.header.actions %}
 {% for action in page.header.actions %}
  <a href="{{ action.url }}">
   <img src="{{ action.img }}" alt="Github">
  </a>
 {% endfor %}
{% endif %}
