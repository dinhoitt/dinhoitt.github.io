---
layout: page
title: "Posts"
permalink: /posts/
main_nav: true
---

<script>
document.addEventListener("DOMContentLoaded", function() {
  var categories = document.querySelectorAll('.category-name');
  categories.forEach(function(cat) {
    cat.addEventListener('click', function() {
      var postsList = this.nextElementSibling;
      if (postsList.style.display === 'none') {
        postsList.style.display = 'block';
      } else {
        postsList.style.display = 'none';
      }
    });
  });
});
</script>

{% for category in site.categories %}
  {% capture cat %}{{ category | first }}{% endcapture %}
  <h2 class="category-name" id="{{cat}}" style="cursor:pointer;">{{ cat | capitalize }}</h2>
  <ul class="posts-list" style="display:none;">
  {% for post in site.categories[cat] %}
    <li>
      <strong>
        <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
      </strong>
      <span class="post-date">- {{ post.date | date_to_long_string }}</span>
    </li>
  {% endfor %}
  </ul>
  {% if forloop.last == false %}<hr>{% endif %}
{% endfor %}
<br>