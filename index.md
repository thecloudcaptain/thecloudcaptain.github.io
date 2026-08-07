---
layout: default
title: Home
description: Practical Azure architecture, migration, networking and infrastructure guidance.
---

<section class="hero">
  <div class="hero-content">
    <span class="hero-badge">
      AZURE · ARCHITECTURE · INFRASTRUCTURE
    </span>

    <h1>
      Navigate the cloud with confidence.
    </h1>

    <p class="hero-description">
      Practical Azure architecture, migration, networking and automation
      guidance built around real-world implementation experience.
    </p>

    <div class="hero-buttons">
      <a class="button button-primary"  href="{{ '/articles/' | relative_url }}">
        Explore articles
      </a>

      <a class="button button-secondary" href="/about/">
        About The Cloud Captain
      </a>
    </div>
  </div>
</section>

<section class="expertise">
  <div class="section-heading">
    <span>WHAT YOU WILL FIND HERE</span>
    <h2>Practical guidance for the Azure journey</h2>
  </div>

  <div class="card-grid">
    <article class="feature-card">
      <div class="card-icon">01</div>

      <h3>Azure Architecture</h3>

      <p>
        Design patterns and platform guidance for secure, scalable and
        resilient Azure environments.
      </p>
    </article>

    <article class="feature-card">
      <div class="card-icon">02</div>

      <h3>Cloud Migration</h3>

      <p>
        Practical migration approaches, proof-of-concept guidance and
        implementation lessons.
      </p>
    </article>

    <article class="feature-card">
      <div class="card-icon">03</div>

      <h3>Infrastructure Automation</h3>

      <p>
        Repeatable deployment patterns using Terraform, PowerShell and
        infrastructure as code.
      </p>
    </article>
  </div>
</section>

<div class="latest-section-background">
  <section class="latest-articles" id="latest-articles">
    <div class="section-heading">
      <span>FROM THE BRIDGE</span>
      <h2>Latest articles</h2>
    </div>

    {% assign latest_post = site.posts.first %}

    {% if latest_post %}
      <article class="article-card latest-article-card{% unless latest_post.image %} latest-article-card-no-image{% endunless %}">
        {% if latest_post.image %}
          <a
            class="latest-article-thumbnail-link"
            href="{{ latest_post.url | relative_url }}"
            aria-label="Read {{ latest_post.title }}"
          >
            <img
              class="latest-article-thumbnail"
              src="{{ latest_post.image | relative_url }}"
              alt="{{ latest_post.image_alt | default: latest_post.title | escape }}"
              loading="lazy"
            >
          </a>
        {% endif %}

        <div class="latest-article-content">
          <div class="article-meta">
            {% if latest_post.categories and latest_post.categories.size > 0 %}
              {{ latest_post.categories | join: " · " | upcase }}
            {% else %}
              CLOUD ARCHITECTURE
            {% endif %}
          </div>

          <h3>
            <a href="{{ latest_post.url | relative_url }}">
              {{ latest_post.title }}
            </a>
          </h3>

          <p>
            {% if latest_post.description %}
              {{ latest_post.description }}
            {% else %}
              {{ latest_post.excerpt | strip_html | strip_newlines | truncate: 240 }}
            {% endif %}
          </p>
        </div>

        <a
          class="article-status article-link latest-article-button"
          href="{{ latest_post.url | relative_url }}"
        >
          Read article →
        </a>
      </article>
    {% else %}
      <article class="article-card latest-article-card latest-article-card-no-image">
        <div class="latest-article-content">
          <div class="article-meta">
            ARTICLES
          </div>

          <h3>
            Technical guidance is being prepared
          </h3>

          <p>
            New Azure architecture, migration, networking and automation
            articles will appear here.
          </p>
        </div>

        <a
          class="article-status article-link latest-article-button"
          href="{{ '/articles/' | relative_url }}"
        >
          View articles →
        </a>
      </article>
    {% endif %}
  </section>
</div>
