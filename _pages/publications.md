---
permalink: /publications/
title: "Publications"
excerpt: "Will High's Publications"
toc: true
toc_sticky: true
author_profile: true
---

I actively published as an academic from 2007 to 2012. 

* {% include linkout.html href="https://arxiv.org/a/high_f_1.html" text="arXiv" %}
* {% include linkout.html href="http://bit.ly/fwhigh-pubs" text="NASA ADS" %}
* {% include linkout.html href="https://inspirehep.net/authors/1053552" text="Insire HEP" %}
* {% include linkout.html href="https://scholar.google.com/citations?user=jR6eVFAAAAAJ&hl=en" text="Google Scholar" %}

## Dissertation

<section class="content-section">
{% for pub in site.data.dissertation %}
<div class="bibliography-item" itemscope itemprop="bibliography" itemtype="http://schema.org/Organization">
  <p class="bibliography-item-year" itemprop="year">{{ pub.year }}</p>
  <p class="bibliography-item-title" itemprop="title">{{ pub.title }}</p>
  <p class="bibliography-item-authors" itemprop="authors">{{ pub.authors }}</p>
  <p class="bibliography-item-publication" itemprop="title">{{ pub.publication }}</p>
  {% if pub.link != "" %}
    <p class="bibliography-item-pdf">{% include linkout.html href=pub.link text="Link" %}</p>
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
    <p class="bibliography-item-pdf">{% include linkout.html href=pub.link text="Link" %}</p>
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
  {% if pub.nasaads != "" %}
    {% capture nasaads_href %}https://ui.adsabs.harvard.edu/abs/{{ pub.nasaads }}/abstract{% endcapture %}
    <p class="bibliography-item-pdf">{% include linkout.html href=nasaads_href text=pub.nasaads %}</p>
  {% endif %}
  {% capture arxiv_href %}https://arxiv.org/abs//{{ pub.arxiv }}{% endcapture %}
  {% capture arxiv_text %}ariXiv:{{ pub.arxiv }}{% endcapture %}
  <p class="bibliography-item-pdf">{% include linkout.html href=arxiv_href text=arxiv_text %}</p>
  {% if pub.download != "" %}
    {% capture pdf_href %}{{ site.url }}{{ site.baseurl }}/assets/publications/{{ pub.download }}{% endcapture %}
    <p class="bibliography-item-pdf">{% include linkout.html href=pdf_href text="PDF" %}</p>
  {% endif %}
</div><!-- end of bibliography-item -->
{% endfor %}
</section>

