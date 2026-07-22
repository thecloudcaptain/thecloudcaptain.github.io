---
layout: default
title: About
permalink: /about/
---

<style>
  /* ======================================================
     About page — scoped styles
     ====================================================== */

  .about-page {
    overflow: hidden;
    color: #071a33;
  }

  .about-container {
    width: min(1120px, calc(100% - 40px));
    margin: 0 auto;
  }

  .about-section {
    padding: 80px 0;
  }

  .about-section-light {
    background: #ffffff;
  }

  .about-section-soft {
    background: #f4f8fc;
  }

  .about-label {
    display: inline-block;
    margin-bottom: 12px;
    color: #0878d1;
    font-size: 0.75rem;
    font-weight: 800;
    letter-spacing: 0.14em;
    text-transform: uppercase;
  }

  .about-heading {
    max-width: 780px;
    margin: 0 0 18px;
    color: #071a33;
    font-size: clamp(2rem, 4vw, 3rem);
    line-height: 1.12;
  }

  .about-intro {
    max-width: 850px;
    margin: 0;
    color: #53677f;
    font-size: 1rem;
    line-height: 1.75;
  }

  .about-intro + .about-intro {
    margin-top: 14px;
  }

  /* Focus areas */

  .about-focus-grid {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 18px;
    margin-top: 38px;
  }

  .about-focus-card {
    position: relative;
    min-height: 210px;
    padding: 28px;
    overflow: hidden;
    background: #ffffff;
    border: 1px solid #dce7f1;
    border-radius: 16px;
    box-shadow: 0 14px 35px rgba(8, 33, 61, 0.07);
  }

  .about-focus-card::after {
    content: "";
    position: absolute;
    right: -34px;
    bottom: -34px;
    width: 100px;
    height: 100px;
    border-radius: 50%;
    background: #e8f4ff;
  }

  .about-focus-number {
    display: inline-flex;
    width: 44px;
    height: 44px;
    align-items: center;
    justify-content: center;
    margin-bottom: 20px;
    border-radius: 11px;
    background: linear-gradient(145deg, #0878d1, #0798eb);
    color: #ffffff;
    font-size: 0.86rem;
    font-weight: 800;
    box-shadow: 0 8px 18px rgba(8, 120, 209, 0.22);
  }

  .about-focus-card h3 {
    position: relative;
    z-index: 1;
    margin: 0 0 10px;
    color: #071a33;
    font-size: 1.08rem;
    line-height: 1.4;
  }

  .about-focus-card p {
    position: relative;
    z-index: 1;
    margin: 0;
    color: #64778d;
    font-size: 0.92rem;
    line-height: 1.65;
  }

  /* Audience */

  .about-audience-panel {
    display: grid;
    grid-template-columns: minmax(0, 1.4fr) minmax(280px, 0.6fr);
    gap: 50px;
    align-items: center;
    padding: 42px;
    background: #ffffff;
    border: 1px solid #dce7f1;
    border-radius: 22px;
    box-shadow: 0 18px 45px rgba(8, 33, 61, 0.07);
  }

  .about-audience-list {
    display: grid;
    gap: 10px;
  }

  .about-audience-item {
    padding: 14px 16px;
    background: #f4f8fc;
    border: 1px solid #e0eaf3;
    border-radius: 12px;
  }

  .about-audience-item strong {
    display: block;
    margin-bottom: 3px;
    color: #071a33;
    font-size: 0.94rem;
  }

  .about-audience-item span {
    color: #64778d;
    font-size: 0.84rem;
    line-height: 1.45;
  }

  /* Author */

  .about-author-layout {
    display: grid;
    grid-template-columns: 280px minmax(0, 1fr);
    gap: 54px;
    align-items: start;
  }

  .about-author-photo-wrap {
    position: relative;
    width: 280px;
  }

  .about-author-photo-wrap::before {
    content: "";
    position: absolute;
    inset: 14px -14px -14px 14px;
    border-radius: 20px;
    background: linear-gradient(145deg, #dcebfa, #eef6fd);
  }

  .about-author-photo {
    position: relative;
    display: block;
    width: 280px;
    height: 350px;
    object-fit: cover;
    object-position: center top;
    border-radius: 20px;
    box-shadow: 0 20px 45px rgba(7, 26, 51, 0.16);
  }

  .about-author-content h2 {
    margin: 0 0 8px;
    color: #071a33;
    font-size: clamp(2.2rem, 4vw, 3.2rem);
    line-height: 1.1;
  }

  .about-author-role {
    margin: 0 0 22px;
    color: #0878d1;
    font-size: 1.04rem;
    font-weight: 750;
    line-height: 1.55;
  }

  .about-author-content p {
    max-width: 760px;
    margin: 0 0 14px;
    color: #53677f;
    font-size: 0.96rem;
    line-height: 1.72;
  }

  .about-author-actions {
    display: flex;
    gap: 16px;
    align-items: center;
    flex-wrap: wrap;
    margin-top: 24px;
  }

  .about-button-primary {
    display: inline-flex;
    min-height: 44px;
    align-items: center;
    justify-content: center;
    padding: 0 20px;
    border-radius: 8px;
    background: #0878d1;
    color: #ffffff;
    font-size: 0.9rem;
    font-weight: 750;
    text-decoration: none;
    transition: transform 0.2s ease, background 0.2s ease;
  }

  .about-button-primary:hover {
    background: #0668b7;
    color: #ffffff;
    transform: translateY(-1px);
  }

  .about-text-link {
    color: #253b55;
    font-size: 0.9rem;
    font-weight: 750;
    text-decoration: none;
  }

  .about-text-link:hover {
    color: #0878d1;
  }

  /* Certifications — compact */

  .about-certification-list {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 16px;
    margin-top: 32px;
  }

  .about-certification-card {
    display: grid;
    grid-template-columns: 44px minmax(0, 1fr);
    gap: 14px;
    align-items: start;
    min-height: 145px;
    padding: 20px;
    background: #ffffff;
    border: 1px solid #dce7f1;
    border-radius: 14px;
    box-shadow: 0 10px 28px rgba(8, 33, 61, 0.055);
  }

  .about-certification-icon {
    display: flex;
    width: 44px;
    height: 44px;
    align-items: center;
    justify-content: center;
    border-radius: 11px;
    background: linear-gradient(145deg, #0878d1, #0798eb);
    color: #ffffff;
    font-size: 0.82rem;
    font-weight: 800;
  }

  .about-certification-content h3 {
    margin: 0 0 7px;
    color: #071a33;
    font-size: 0.94rem;
    line-height: 1.4;
  }

  .about-certification-content p {
    margin: 0;
    color: #64778d;
    font-size: 0.8rem;
    line-height: 1.5;
  }

  .about-certification-status {
    display: inline-block;
    margin-top: 9px;
    color: #687b90;
    font-size: 0.72rem;
    font-weight: 750;
  }

  .about-certification-status-active {
    color: #0878d1;
  }

  /* Final CTA */

  .about-cta-panel {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 30px;
    padding: 38px 42px;
    border-radius: 20px;
    background: linear-gradient(120deg, #061d36, #075da7);
    color: #ffffff;
  }

  .about-cta-panel .about-label {
    color: #4fc3ff;
  }

  .about-cta-panel h2 {
    max-width: 700px;
    margin: 0;
    color: #ffffff;
    font-size: clamp(1.65rem, 3vw, 2.35rem);
    line-height: 1.25;
  }

  .about-cta-actions {
    display: flex;
    gap: 12px;
    flex-shrink: 0;
  }

  .about-button-light,
  .about-button-outline {
    display: inline-flex;
    min-height: 42px;
    align-items: center;
    justify-content: center;
    padding: 0 18px;
    border-radius: 8px;
    font-size: 0.86rem;
    font-weight: 750;
    text-decoration: none;
  }

  .about-button-light {
    background: #ffffff;
    color: #071a33;
  }

  .about-button-outline {
    border: 1px solid rgba(255, 255, 255, 0.45);
    color: #ffffff;
  }

  /* Responsive */

  @media (max-width: 900px) {
    .about-focus-grid,
    .about-certification-list {
      grid-template-columns: repeat(2, minmax(0, 1fr));
    }

    .about-audience-panel {
      grid-template-columns: 1fr;
      gap: 28px;
    }

    .about-author-layout {
      grid-template-columns: 230px minmax(0, 1fr);
      gap: 38px;
    }

    .about-author-photo-wrap,
    .about-author-photo {
      width: 230px;
    }

    .about-author-photo {
      height: 290px;
    }

    .about-cta-panel {
      align-items: flex-start;
      flex-direction: column;
    }
  }

  @media (max-width: 650px) {
    .about-container {
      width: min(100% - 28px, 1120px);
    }

    .about-section {
      padding: 58px 0;
    }

    .about-focus-grid,
    .about-certification-list {
      grid-template-columns: 1fr;
    }

    .about-audience-panel {
      padding: 28px 22px;
    }

    .about-author-layout {
      grid-template-columns: 1fr;
      gap: 38px;
    }

    .about-author-photo-wrap {
      width: 210px;
    }

    .about-author-photo {
      width: 210px;
      height: 265px;
    }

    .about-certification-card {
      min-height: auto;
    }

    .about-cta-panel {
      padding: 28px 22px;
    }

    .about-cta-actions {
      flex-wrap: wrap;
    }
  }
</style>


<div class="about-page">

  <section class="hero">
    <div class="hero-content">

      <div class="hero-badge">
        ABOUT THE CLOUD CAPTAIN
      </div>

      <h1>
        Practical cloud guidance built around real-world delivery.
      </h1>

      <p>
        The Cloud Captain is a cloud strategy, architecture and engineering
        platform focused on helping organisations design, modernise and operate
        secure, resilient, scalable and cost-efficient cloud environments.
      </p>

      <div class="hero-actions">
        <a href="/articles/" class="button button-primary">
          Explore articles
        </a>

        <a href="#about-author" class="button button-secondary">
          About the author
        </a>
      </div>

    </div>
  </section>


  <section class="about-section about-section-light">
    <div class="about-container">

      <span class="about-label">
        Our purpose
      </span>

      <h2 class="about-heading">
        Bridging cloud strategy and implementation
      </h2>

      <p class="about-intro">
        The Cloud Captain shares practical, implementation-focused guidance
        based on real-world enterprise cloud challenges. Its purpose is to
        connect high-level strategy with the architectural and engineering
        decisions required to deliver secure, reliable and production-ready
        solutions.
      </p>

      <p class="about-intro">
        The platform covers established areas of cloud architecture and
        infrastructure while also documenting the evolving journey into
        artificial intelligence.
      </p>


      <div class="about-focus-grid">

        <article class="about-focus-card">
          <div class="about-focus-number">01</div>
          <h3>Azure Architecture</h3>
          <p>
            Azure infrastructure architecture, cloud modernisation, platform
            design and practical guidance for complex enterprise workloads.
          </p>
        </article>

        <article class="about-focus-card">
          <div class="about-focus-number">02</div>
          <h3>Networking and Hybrid DNS</h3>
          <p>
            Enterprise networking, hybrid connectivity, private access,
            DNS forwarding and hybrid name-resolution architecture.
          </p>
        </article>

        <article class="about-focus-card">
          <div class="about-focus-number">03</div>
          <h3>Migration and Modernisation</h3>
          <p>
            Migration strategy, proof-of-concept delivery, technical
            validation and practical cloud adoption guidance.
          </p>
        </article>

        <article class="about-focus-card">
          <div class="about-focus-number">04</div>
          <h3>Automation and Infrastructure as Code</h3>
          <p>
            Repeatable infrastructure and platform engineering patterns using
            Terraform, Bicep, PowerShell and automation.
          </p>
        </article>

        <article class="about-focus-card">
          <div class="about-focus-number">05</div>
          <h3>Cost Optimisation and FinOps</h3>
          <p>
            Azure cost management, governance and FinOps practices aligned
            with business and operational requirements.
          </p>
        </article>

        <article class="about-focus-card">
          <div class="about-focus-number">06</div>
          <h3>AI Learning Journey</h3>
          <p>
            Exploring AI architecture, Azure AI services and practical use
            cases while navigating the evolving AI journey together.
          </p>
        </article>

      </div>

    </div>
  </section>


  <section class="about-section about-section-soft">
    <div class="about-container">

      <div class="about-audience-panel">

        <div>
          <span class="about-label">
            Who the platform is for
          </span>

          <h2 class="about-heading">
            Guidance for people delivering cloud transformation
          </h2>

          <p class="about-intro">
            The Cloud Captain is built for cloud architects, infrastructure
            engineers, platform teams, technology leaders and organisations
            looking for clear, practical and implementation-focused guidance.
          </p>

          <p class="about-intro">
            Whether you are designing an Azure platform, modernising existing
            infrastructure, solving hybrid connectivity challenges, improving
            governance, managing cloud cost or beginning your AI journey, the
            objective is to provide guidance that can be applied in the real
            world.
          </p>
        </div>


        <div class="about-audience-list">

          <div class="about-audience-item">
            <strong>Cloud architects</strong>
            <span>Designing secure and scalable cloud platforms</span>
          </div>

          <div class="about-audience-item">
            <strong>Infrastructure engineers</strong>
            <span>Implementing connectivity, compute and automation</span>
          </div>

          <div class="about-audience-item">
            <strong>Technology leaders</strong>
            <span>Planning modernisation and transformation</span>
          </div>

          <div class="about-audience-item">
            <strong>Cloud and AI learners</strong>
            <span>Developing practical technical knowledge</span>
          </div>

        </div>

      </div>

    </div>
  </section>


  <section
    class="about-section about-section-light"
    id="about-author">

    <div class="about-container">

      <div class="about-author-layout">

        <div class="about-author-photo-wrap">

          <img
            src="/assets/images/usman-mahmood-profile.png"
            alt="Usman Mahmood, founder of The Cloud Captain"
            class="about-author-photo">

        </div>


        <div class="about-author-content">

          <span class="about-label">
            About the author
          </span>

          <h2>
            Usman Mahmood
          </h2>

          <p class="about-author-role">
            Cloud Solution Architect specialising in Azure infrastructure,
            networking and enterprise cloud transformation.
          </p>

          <p>
            Usman Mahmood is an experienced cloud and infrastructure architect
            who helps organisations translate business and technology
            objectives into secure, resilient and operationally sustainable
            cloud solutions.
          </p>

          <p>
            He currently works at Microsoft as a Cloud Solution Architect,
            focusing on Azure infrastructure and complex enterprise
            transformation initiatives. His career has developed across
            enterprise IT, infrastructure and telecommunications before
            progressing into cloud architecture and strategic technology
            advisory.
          </p>

          <p>
            His experience spans cloud strategy, enterprise architecture,
            infrastructure modernisation, hybrid-cloud design and the delivery
            of complex transformation programmes. His work covers technical
            discovery, architecture development, proof-of-concept validation,
            migration planning, platform governance, resilience and
            operational readiness.
          </p>

          <p>
            His specialist areas include Azure infrastructure architecture,
            enterprise networking, private connectivity, hybrid DNS, Azure
            landing zones, cloud governance, migration strategy, business
            continuity, disaster recovery, Infrastructure as Code and Azure
            cost optimisation.
          </p>

          <p>
            Through The Cloud Captain, Usman shares practical architecture
            guidance, implementation lessons and technical insights shaped by
            real-world delivery experience. He is also documenting his
            learning journey across AI architecture, Azure AI services and
            enterprise AI adoption.
          </p>

          <div class="about-author-actions">

            <a
              href="https://www.linkedin.com/in/usman-mahmood-profile/"
              class="about-button-primary"
              target="_blank"
              rel="noopener noreferrer">
              Connect on LinkedIn
            </a>

            <a href="/articles/" class="about-text-link">
              Read published articles →
            </a>

          </div>

        </div>

      </div>

    </div>
  </section>


  <section class="about-section about-section-soft">
    <div class="about-container">

      <span class="about-label">
        Professional certifications
      </span>

      <h2 class="about-heading">
        Microsoft Azure certifications
      </h2>

      <p class="about-intro">
        Credentials supporting Usman’s experience across Azure architecture,
        administration and emerging AI technologies.
      </p>


      <div class="about-certification-list">

        <article class="about-certification-card">

          <div class="about-certification-icon">
            AI
          </div>

          <div class="about-certification-content">
            <h3>
              Microsoft Certified: Azure AI Fundamentals
            </h3>

            <p>
              Foundational knowledge of artificial intelligence, machine
              learning and Azure AI services.
            </p>

            <span
              class="about-certification-status
                     about-certification-status-active">
              Active certification
            </span>
          </div>

        </article>


        <article class="about-certification-card">

          <div class="about-certification-icon">
            AZ
          </div>

          <div class="about-certification-content">
            <h3>
              Microsoft Certified: Azure Solutions Architect Expert
            </h3>

            <p>
              Expert-level knowledge across Azure infrastructure, identity,
              governance, resilience and enterprise solution design.
            </p>

            <span class="about-certification-status">
              Previously held certification
            </span>
          </div>

        </article>


        <article class="about-certification-card">

          <div class="about-certification-icon">
            AD
          </div>

          <div class="about-certification-content">
            <h3>
              Microsoft Certified: Azure Administrator Associate
            </h3>

            <p>
              Azure administration across identity, governance, networking,
              compute, storage and operational management.
            </p>

            <span class="about-certification-status">
              Previously held certification
            </span>
          </div>

        </article>

      </div>

    </div>
  </section>


  <section class="about-section about-section-light">
    <div class="about-container">

      <div class="about-cta-panel">

        <div>
          <span class="about-label">
            From the bridge
          </span>

          <h2>
            Explore practical cloud architecture and engineering guidance
          </h2>
        </div>

        <div class="about-cta-actions">

          <a href="/articles/" class="about-button-light">
            Explore articles
          </a>

          <a href="/" class="about-button-outline">
            Back to home
          </a>

        </div>

      </div>

    </div>
  </section>

</div>
