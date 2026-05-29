---
layout: home
title: Home
---

Arun Samayam

## Latest Post

{% assign latest_post = site.posts.first %}
{% if latest_post %}
### [{{ latest_post.title }}]({{ latest_post.url | relative_url }})

{{ latest_post.date | date: "%B %-d, %Y" }}

{{ latest_post.description | default: latest_post.excerpt }}
{% endif %}