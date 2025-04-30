---
title: Automate Azure Load Testing with GitHub Actions
permalink: index.html
layout: home
---

The following exercises are designed to provide you with a hands-on learning experience implementing GitHub actions and workflows to automate performing a load test with Azure Load Testing. 

## Exercises
<hr/>


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
{% for activity in labs  %}
* [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) <br/> {{ activity.lab.description }}
{% endfor %}
