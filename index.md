--- 

layout: home
---
<h2>Latest posts </h2>
{% for post in site.posts %}
  <article>
    <h3>
      <a href="{{ post.url }}">
        {{ post.title }}
      </a>
    </h3>
    <p class="post-excerpt">
    {% if post.content contains '<!--excerpt.start-->' and post.content contains '<!--excerpt.end-->' %}
    	{{ ((post.content | split:'<!--excerpt.start-->' | last) | split: '<!--excerpt.end-->' | first) | strip_html | truncatewords: 20 }}
    {% else %}
    	{{ post.content | strip_html | truncatewords: 30 }}
        {% endif %}
        </p>
        <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time> 
  </article>
  <hr>
{% endfor %}

<h2>About me </h2>
My name is Jack and I'm the lead integrations engineer at [Adaptivity](https://adaptivity.uk/), which is an integrations specialist for building APIs and connecting systems.

This is a blog for some general advice and helpful how-to guides within the Azure platform. Most of the posts are discussing Azures commonly used integrations products such as logic apps, function apps and kubernetes. 

You can reach me on my email (jack.weldon@adaptivity.uk)
or my [linkedIn](https://www.linkedin.com/in/jackweldon/) profile 

<div data-iframe-width="150" data-iframe-height="270" data-share-badge-id="220078a9-703c-4645-a83a-1c3968886f89" data-share-badge-host="https://www.youracclaim.com"></div><script type="text/javascript" async src="//cdn.youracclaim.com/assets/utilities/embed.js"></script>