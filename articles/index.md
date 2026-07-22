---
layout: default
title: Articles
description: Practical articles covering Azure architecture, cloud infrastructure, migration, networking, security and engineering.
permalink: /articles/
---

<section class="page-hero articles-hero">
  <div class="page-container">
    <span class="eyebrow">Insights and implementation guidance</span>

    <h1>Cloud Architecture & Engineering Articles</h1>

    <p>
      Practical, implementation-focused guidance covering Azure infrastructure,
      cloud migration, enterprise networking, security, governance and
      operational resilience.
    </p>
  </div>
</section>

<section class="articles-section">
  <div class="page-container">

    <div class="section-heading">
      <span class="eyebrow">Latest publications</span>
      <h2>Featured Articles</h2>

      <p>
        Detailed technical guidance developed around real-world cloud
        architecture and infrastructure challenges.
      </p>
    </div>

    <div class="articles-grid">

      {% assign published_posts = site.posts %}

      {% if published_posts.size > 0 %}

        {% for post in published_posts %}

          <article class="article-card">

            <div class="article-card-top">
              <span class="article-category">
                {{ post.category | default: "Azure Architecture" }}
              </span>

              {% if post.status %}
                <span class="article-status">
                  {{ post.status }}
                </span>
              {% endif %}
            </div>

            <h3>
              <a href="{{ post.url | relative_url }}">
                {{ post.title }}
              </a>
            </h3>

            <p>
              {% if post.description %}
                {{ post.description }}
              {% elsif post.excerpt %}
                {{ post.excerpt | strip_html | truncate: 220 }}
              {% else %}
                Practical cloud architecture and engineering guidance from
                The Cloud Captain.
              {% endif %}
            </p>

            <div class="article-card-footer">
              <span class="article-date">
                {{ post.date | date: "%d %B %Y" }}
              </span>

              <a
                class="article-link"
                href="{{ post.url | relative_url }}"
                aria-label="Read {{ post.title }}"
              >
                Read article
                <span aria-hidden="true">&rarr;</span>
              </a>
            </div>

          </article>

        {% endfor %}

      {% else %}

        <article class="article-card">

          <div class="article-card-top">
            <span class="article-category">
              Azure Architecture
            </span>

            <span class="article-status">
              Coming soon
            </span>
          </div>

          <h3>Technical articles are being prepared</h3>

          <p>
            New implementation-focused guidance covering Azure architecture,
            infrastructure, migration, networking and security will appear
            here.
          </p>

        </article>

      {% endif %}

    </div>

  </div>
</section>
