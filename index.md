---
Layout: default
Title: LYJ's blog
---



<h1>{{page.title}}</h1>

<ul class="posts">
  
</ul>

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>







