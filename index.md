---
layout: default
title: Home
description: Cybersecurity portfolio, certifications, projects, and technical writeups by Niko Gonzalez.
---

<section class="hero">
  <div class="container hero-layout">
    <div class="hero-mark">
      <img
        class="hero-logo"
        src="{{ '/assets/images/rambosec-logo-white.png' | relative_url }}"
        alt="RamboSec logo"
      >
    </div>

    <div class="hero-copy-wrap">
      <p class="eyebrow">Cybersecurity Analyst | Threat Hunting | Security Investigations</p>
      <h1>Nicholas Gonzalez</h1>
      <p class="hero-copy">
        RamboSec is my cybersecurity portfolio and research site. It brings together certifications, technical writing, projects, and ongoing learning across digital forensics, threat hunting, security investigations, and the broader field of cybersecurity.
      </p>

      <div class="hero-actions">
        <a class="button button-primary" href="{{ '/certifications/' | relative_url }}">Certifications</a>
        <a class="button" href="{{ '/blog/' | relative_url }}">Blog</a>
        <a class="button" href="{{ '/about/' | relative_url }}">About</a>
      </div>
    </div>
  </div>
</section>

<section class="section-band">
  <div class="container">
    <div class="section-heading">
      <div>
        <p class="eyebrow">Overview</p>
        <h2>What you will find here.</h2>
      </div>
      <p class="muted">
        This site is a place to collect and present my certifications, technical writeups, and hands on projects in a way that is clear, professional, and easy to review.
      </p>
    </div>

    <div class="card-grid three-up">
      <article class="panel card">
        <p class="card-kicker">01</p>
        <h3>Certifications</h3>
        <p>A growing record of completed certifications and technical milestones across cybersecurity and related areas.</p>
      </article>

      <article class="panel card">
        <p class="card-kicker">02</p>
        <h3>Writeups</h3>
        <p>Posts covering exam experiences, technical notes, walkthroughs, and other topics that support ongoing learning.</p>
      </article>

      <article class="panel card">
        <p class="card-kicker">03</p>
        <h3>Projects and Labs</h3>
        <p>Hands on work, experiments, and practical exploration across different areas of cybersecurity.</p>
      </article>
    </div>
  </div>
</section>

<section class="section-shell">
  <div class="container">
    <div class="section-heading">
      <div>
        <p class="eyebrow">Recent Posts</p>
        <h2>Latest writing</h2>
      </div>
      <a class="text-link" href="{{ '/blog/' | relative_url }}">View all posts</a>
    </div>

    <div class="card-grid">
      {% for post in site.posts limit:3 %}
      <article class="panel post-card">
        <p class="post-card-date">{{ post.date | date: "%B %-d, %Y" }}</p>
        <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
        <p>{{ post.description | default: post.excerpt | strip_html | truncate: 150 }}</p>
      </article>
      {% endfor %}
    </div>
  </div>
</section>

<section class="section-shell">
  <div class="container">
    <div class="section-heading">
      <div>
        <p class="eyebrow">Current Interests</p>
        <h2>Areas currently represented on the site.</h2>
      </div>
      <p class="muted">
        RamboSec covers a growing mix of investigations, certifications, lab work, and technical learning across cybersecurity and adjacent computing topics. Some areas are already represented in depth, while others are actively being built out.
      </p>
    </div>

    <div class="card-grid">
      <article class="panel card">
        <h3>Security Investigations</h3>
        <p>Practical analysis of suspicious activity, alerts, and investigative findings.</p>
      </article>

      <article class="panel card">
        <h3>Threat Hunting</h3>
        <p>Exploration of telemetry, indicators, and patterns relevant to defensive security work.</p>
      </article>

      <article class="panel card">
        <h3>Digital Forensics</h3>
        <p>Posts and projects related to evidence review, artifact analysis, and investigative workflows.</p>
      </article>

      <article class="panel card">
        <h3>Certifications and Learning</h3>
        <p>Exam preparation, study reflections, and technical growth over time.</p>
      </article>

      <article class="panel card">
        <h3>Labs and Homelab</h3>
        <p>Hands on setup, experiments, tooling, and self directed practice.</p>
      </article>

      <article class="panel card">
        <h3>Writeups and Walkthroughs</h3>
        <p>CTFs, technical notes, and other learning focused posts across a range of topics.</p>
      </article>
    </div>
  </div>
</section>

<section class="section-shell compact-top">
  <div class="container two-column-callout">
    <div class="panel">
      <p class="eyebrow">Professional Links</p>
      <h2>Connect and review.</h2>
      <ul class="clean-list">
        {% for entry in site.data.social %}
        <li><a href="{{ entry[1].url }}">{{ entry[1].label }}</a></li>
        {% endfor %}
      </ul>
    </div>

    <div class="panel">
      <p class="eyebrow">Resume</p>
      <h2>Available upon request.</h2>
      <p>
        A current resume is available for recruiters and hiring teams upon request. Please reach out at <a href="mailto:resume@rambosec.com">resume@rambosec.com</a>.
      </p>
      <a class="text-link" href="{{ '/resume/' | relative_url }}">Resume request details</a>
    </div>
  </div>
</section>
