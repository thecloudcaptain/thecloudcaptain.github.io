---
layout: default
title: Articles
description: Practical cloud architecture, Azure infrastructure, migration, networking, security and engineering guidance.
permalink: /articles/
---

<section class="articles-hero">
  <div class="site-container">
    <span class="section-label">INSIGHTS &amp; IMPLEMENTATION GUIDANCE</span>

    <h1>Cloud Architecture &amp; Engineering Articles</h1>

    <p>
      Practical, implementation-focused guidance covering Azure infrastructure,
      cloud migration, enterprise networking, security, governance and
      operational resilience.
    </p>
  </div>
</section>

<section class="articles-listing">
  <div class="site-container">

    <div class="articles-heading">
      <span class="section-label">LATEST PUBLICATIONS</span>

      <h2>Featured Articles</h2>

      <p>
        Detailed technical guidance developed around real-world cloud
        architecture and infrastructure challenges.
      </p>
    </div>

    {% if site.posts.size > 0 %}

      <div class="articles-grid">

        {% for post in site.posts %}

          <article class="article-card">

            <div class="article-card-content">

              <div class="article-card-meta">
                <span class="article-category">
                  {% if post.categories and post.categories.size > 0 %}
                    {{ post.categories | join: " · " }}
                  {% else %}
                    Cloud Architecture
                  {% endif %}
                </span>

                <span class="article-date">
                  {{ post.date | date: "%d %B %Y" }}
                </span>
              </div>

              <h3>
                <a href="{{ post.url | relative_url }}">
                  {{ post.title }}
                </a>
              </h3>

              <p class="article-description">
                {% if post.description %}
                  {{ post.description }}
                {% else %}
                  {{ post.excerpt | strip_html | strip_newlines | truncate: 240 }}
                {% endif %}
              </p>

              <a
                class="article-read-link"
                href="{{ post.url | relative_url }}"
                aria-label="Read {{ post.title }}"
              >
                Read article
                <span aria-hidden="true">&rarr;</span>
              </a>

            </div>

          </article>

        {% endfor %}

      </div>

    {% else %}

      <div class="articles-empty-state">
        <span class="section-label">COMING SOON</span>

        <h3>Technical articles are being prepared</h3>

        <p>
          New implementation-focused guidance covering Azure architecture,
          infrastructure, migration, networking and security will be published
          here.
        </p>
      </div>

    {% endif %}

  </div>
</section>
