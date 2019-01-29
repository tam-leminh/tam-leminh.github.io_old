---
bg: "owl.jpg"
layout: page
title: "Projects"
crawlertitle: "Projects - Tâm Le Minh"
permalink: /projects/
summary: "Some of my projects in Software or AI"
active: projects
---

<div>
{% for post in site.posts %}
  {% if post.type == "project" %}

<div style="text-align: center" class="inline-block">
  <a href="{{ post.url | relative_url }}">
    <img class="project-round" src="{{ site.images | relative_url }}/{{ post.bg }}" />
  </a>
  <div>
  <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
  </div>
  <div>
  <a target="_blank" style="background-color: olivedrab; color: white" href="{{ post.url | relative_url }}" class="linkbutton">Article</a>
  <a target="_blank" style="background-color: saddlebrown; color: white" href="{{ post.github }}" class="linkbutton">Github</a>
  </div>
</div>

  {% endif %}
{% endfor %}
</div>

{% for post in site.posts %}
  {% if post.type == "project" %}

<article class="index-page">
  <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
  {{ post.excerpt }}
</article>
	
  {% endif %}
{% endfor %}
