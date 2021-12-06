---
layout: cat
---


{% assign sortedFiches = site.fiches | sort: 'title' %}
  <h1>Fiches</h1>
  <ul>
    {% for fiche in sortedFiches %}
      {% unless fiche.subelement %}
      <li><a href="{{ site.baseurl }}/{{ fiche.url }}">{{ fiche.title }}</a></li>
      {% endunless %}
    {% endfor %}
  </ul>
