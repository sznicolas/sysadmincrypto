---
layout: fiche
title:  "Scalability Layers"
date:   2021-03-25 
categories: fiches 
cat: scalability_layer
permalink: scalability
---


<ul>
  {% for fiche in site.fiches %}
    {% if fiche.categories contains page.cat  %}
      <li><a href="{{site.baseurl}}/{{ fiche.url }}">{{ fiche.title }}{{ fiche.excerpt }}</a></li>
    {% endif %}
  {% endfor %}
</ul>
