---
layout: default

permalink: /projects/
pagination:
  enabled: true
  tag: project
---

<div class="page" markdown="1">
  <div class="page-title-container">
    <div class="heading-container">
      <h1>Projects</h1>
      <span>A collection of personal projects I've worked on since starting this blog.</span>
    </div>
  </div>
</div>

{% for post in paginator.posts %}
  {% include blog.html %}
{% endfor %}

{% if paginator.total_pages > 1 %}
<nav class="pagination" role="navigation">
  <hr/>
    <ul>
      <li class="pagination-item-prev" >
            {% if paginator.previous_page %}
        {% if paginator.page == 2 %}
              <a rel="prev" href="{{ site.baseurl }}/">&laquo; Previous</a>
          {% else %}
          <a rel="prev" href="{{ site.baseurl }}/page/{{paginator.previous_page}}/">&laquo; Previous</a>
          {% endif %}
        {% else %}
        <span>&laquo; Previous</span>
            {% endif %}
      </li>
  
  
      <li class="pagination-item-next" >
            {% if paginator.next_page %}
        <a rel="next" href="{{ site.baseurl }}/{{paginator.next_page}}/">Next &raquo;</a>
        {% else %}
        <span>Next &raquo;</span>
            {% endif %}
      </li>
  
    </ul>
  <hr/>
</nav>
{% endif %}