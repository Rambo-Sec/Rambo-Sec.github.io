---
layout: page
title: Certifications & Licenses
permalink: /certifications/
description: Professional certifications, licenses, and education.
---

<p>
  This page collects current certifications, licenses, and related credentials. Where available, entries include issue date, expiration date, credential ID, and a verification link.
</p>

<div class="cert-grid">
  {% for cert in site.data.certifications %}
  <article class="panel cert-card">
    <div class="cert-card-top">
      <p class="cert-status {% if cert.status == 'Expired' %}is-expired{% endif %}">{{ cert.status }}</p>
      <h2>{{ cert.name }}</h2>
      <p class="issuer">{{ cert.issuer }}</p>
    </div>

    <dl class="meta-list">
      {% if cert.issued and cert.issued != "" %}
      <div>
        <dt>Issued</dt>
        <dd>{{ cert.issued }}</dd>
      </div>
      {% endif %}

      {% if cert.expires and cert.expires != "" %}
      <div>
        <dt>Expires</dt>
        <dd>{{ cert.expires }}</dd>
      </div>
      {% endif %}

      {% if cert.credential_id and cert.credential_id != "" %}
      <div>
        <dt>Credential ID</dt>
        <dd>{{ cert.credential_id }}</dd>
      </div>
      {% endif %}
    </dl>

    {% if cert.verify_url and cert.verify_url != "" %}
    <p class="cert-link">
      <a class="text-link" href="{{ cert.verify_url }}" target="_blank" rel="noopener noreferrer">Verify credential</a>
    </p>
    {% endif %}
  </article>
  {% endfor %}
</div>
