---
permalink: /publications/
title: "Publications"
excerpt: "Will High's Publications"
last_modified_at: 2021-05-21T00:00:00-07:00
author_profile: true
classes: wide
---

I actively published as an academic from 2007 to 2013.

## Aggregators

<a href="http://bit.ly/fwhigh-arxiv" target="_blank">arXiv</a> <i class="fas fa-fw fa-external-link-alt"></i>

<a href="http://bit.ly/fwhigh-pubs" target="_blank">NASA ADS</a> <i class="fas fa-fw fa-external-link-alt"></i>

<a href="https://inspirehep.net/authors/1053552" target="_blank">Insire HEP</a> <i class="fas fa-fw fa-external-link-alt"></i>

<a href="https://scholar.google.com/citations?user=jR6eVFAAAAAJ&hl=en" target="_blank">Google Scholar</a><i class="fas fa-fw fa-external-link-alt"> </i>

## Dissertation

<section class="content-section">
{% for pub in site.data.dissertation %}
<div class="bibliography-item" itemscope itemprop="bibliography" itemtype="http://schema.org/Organization">
  <p class="bibliography-item-year" itemprop="year">{{ pub.year }}</p>
  <p class="bibliography-item-title" itemprop="title">{{ pub.title }}</p>
  <p class="bibliography-item-authors" itemprop="authors">{{ pub.authors }}</p>
  <p class="bibliography-item-publication" itemprop="title">{{ pub.publication }}</p>
  {% if pub.link != "" %}
	  <p class="bibliography-item-pdf"><a href="{{ pub.link }}">Link</a> <i class="fas fa-fw fa-external-link-alt"></i></p>
  {% endif %}
</div><!-- end of bibliography-item -->
{% endfor %}
</section>

## Grants

<section class="content-section">
{% for pub in site.data.grants %}
<div class="bibliography-item" itemscope itemprop="bibliography" itemtype="http://schema.org/Organization">
  <p class="bibliography-item-year" itemprop="year">{{ pub.year }}</p>
  <p class="bibliography-item-title" itemprop="title">{{ pub.title }}</p>
  <p class="bibliography-item-authors" itemprop="authors">{{ pub.authors }}</p>
  <p class="bibliography-item-publication" itemprop="title">{{ pub.publication }}</p>
  {% if pub.link != "" %}
	  <p class="bibliography-item-pdf"><a href="{{ pub.link }}">Link</a> <i class="fas fa-fw fa-external-link-alt"></i></p>
  {% endif %}
</div><!-- end of bibliography-item -->
{% endfor %}
</section>

## Bibliography

<section class="content-section">
{% for pub in site.data.bibliography %}
<div class="bibliography-item" itemscope itemprop="bibliography" itemtype="http://schema.org/Organization">
  <p class="bibliography-item-year" itemprop="year">{{ pub.year }}</p>
  <p class="bibliography-item-title" itemprop="title">{{ pub.title }}</p>
  <p class="bibliography-item-authors" itemprop="authors">{{ pub.authors }}</p>
  <p class="bibliography-item-publication" itemprop="title">{{ pub.publication }}</p>
  <p class="bibliography-item-arxiv"><a href="https://arxiv.org/abs//{{ pub.arxiv }}" target="_blank">ariXiv:{{ pub.arxiv }}</a> <i class="fas fa-fw fa-external-link-alt"></i></p>
  {% if pub.download != "" %}
	  <p class="bibliography-item-pdf"><a href="{{ site.url }}{{ site.baseurl }}/assets/publications/{{ pub.download }}" target="_blank">PDF</a> <i class="fas fa-fw fa-external-link-alt"></i></p>
  {% endif %}
</div><!-- end of bibliography-item -->
{% endfor %}
</section>

