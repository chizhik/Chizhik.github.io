---
layout: page
title: Projects
---

## List of projects

{% for post in site.posts %}
  * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
{% endfor %}

Other projects can be found on [github](https://github.com/Chizhik)