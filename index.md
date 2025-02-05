---
title: Online gehostete Anweisungen
permalink: index.html
layout: home
---

# Azure SQL-Entwicklungsübungen

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %} [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) {% endfor %}


