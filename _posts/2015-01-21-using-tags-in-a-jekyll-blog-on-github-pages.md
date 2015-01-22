---
description: "Tags are normally implemented in a Jekyll site using Ruby plugins. These don't work on Github pages. This implementation will work without plugins."
tags: 
  - tools
  - web development
  - featured
---

Most dynamic content management systems (e.g. [WordPress](http://wordpress.com)  allows you to easily build tag and category pages that assist in navigation of your site, and add some Google juice.

Unfortunately this isn't so easy if you want to host a static site on [Github Pages](http://pages.github.com).... It's 100% possible if you are generating the site locally and pushing to Github, but unfortunately you can't use Ruby plugins with Github pages.

However, lucky for us, it is possible to hack together a bit of a solution using straight Liquid tags. You can get a whole tags page going that links internally, you just can't generate individual tag pages for each tag.  Can't have it all I suppose.

{% highlight html %}{% raw %}
---
layout: page
title: Tags
---

<div class='list-group'>
  {% assign tags_list = site.tags %}

  {% if tags_list.first[0] == null %}
    {% for tag in tags_list %}
      <a href="/tags#{{ tag }}-ref" class='list-group-item'>
        {{ tag }} <span class='badge'>{{ site.tags[tag].size }}</span>
      </a>
    {% endfor %}
  {% else %}
    {% for tag in tags_list %}
      <a href="/tags#{{ tag[0] }}-ref" class='list-group-item'>
        {{ tag[0] }} <span class='badge'>{{ tag[1].size }}</span>
      </a>
    {% endfor %}
  {% endif %}

  {% assign tags_list = nil %}
</div>


{% for tag in site.tags %}
  <h2 class='tag-header' id="{{ tag[0] }}-ref">{{ tag[0] }}</h2>
  <ul>
    {% assign pages_list = tag[1] %}

    {% for node in pages_list %}
      {% if node.title != null %}
        {% if group == null or group == node.group %}
          {% if page.url == node.url %}
          <li class="active"><a href="{{node.url}}" class="active">{{node.title}}</a></li>
          {% else %}
          <li><a href="{{node.url}}">{{node.title}}</a></li>
          {% endif %}
        {% endif %}
      {% endif %}
    {% endfor %}

    {% assign pages_list = nil %}
    {% assign group = nil %}
  </ul>
{% endfor %}
{% endraw %}{% endhighlight %}

There is a live example up on my site at the [tag page](/tags/).
