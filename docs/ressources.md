---
#layout: default
layout: cat
categories: fiches
permalink: ressources
---


<ul>
  {% for ressource in site.ressources %}
    <li><a href="{{ site.baseurl }}/{{ ressource.url }}">{{ ressource.title }}</a></li>
  {% endfor %}
</ul>
