---
layout: page
---


Welcome to Green's Opinion!  Below you'll see some of my projects, open source and recent blog posts:

<div class="row project-listing">
  <div class="col-sm-6 col-md-6 project-item">
    <h3 class="project-title"><a href="http://www.epicrideweather.com">Epic Ride Weather</a></h3>
    <span class="project-image pull-left">
      <a href="http://www.epicrideweather.com"><img src="images/epicrideweather/logo.png" class="img-bordered" alt="Epic Ride Weather app"/></a>
    </span>
    <p>
      <a href="http://www.epicrideweather.com">
      A unique weather app for cyclists that integrates with Strava, Ride With GPS, Komoot and others.
      </a>
    </p>
  </div>
  <div class="col-sm-6 col-md-6 project-item">
    <h3 class="project-title"><a href="https://wiki.eclipse.org/Mylyn/WikiText">WikiText</a></h3>
    <span class="project-image pull-left"><a href="https://wiki.eclipse.org/Mylyn/WikiText"><i class="fa fa-code code-icon" aria-hidden="true"></i></a></span>
    <p>
      <a href="https://wiki.eclipse.org/Mylyn/WikiText">A software library and tools for for transforming and rendering wiki markup, rich text and HTML.</a>
    </p>
  </div>  
</div>

<ul class="post-listing">
{% for post in site.posts limit:6 %}
<li>
  <h3 class="post-title"><a href="{{ post.url }}">{{ post.title }}</a></h3>
  <p>
  {% if post.listing_image %}
  <span class="feature-image pull-left"><a href="{{ post.url }}"><img src="{{post.listing_image}}"/></a></span>
  {% endif %}
  {{ post.content | strip_html | truncatewords:80 }}  <a href="{{ post.url }}" class="text-primary">continue</a>
  </p>
</li>
{% endfor %}
<li><h3 class="post-title"><a href="{{ "/archive" | prepend: site.baseurl }}">All Articles</a></h3></li>
</ul>

