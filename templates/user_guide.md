# User Guide: Prompt-Based SEO Template Generation

## Overview
This guide shows you how to generate a bespoke website template for any business niche using the **template-prompt system**. The workflow consists of:
1. Selecting a niche.
2. Running a series of prompts to create content, SEO metadata, and layout.
3. Building the final static site with the provided scripts.
4. Deploying to the leasing platform.

All prompts are designed to work with Claude/Fable (OpenRouter) but can be used with any LLM.

---

## 1. Prompt Library

### 1.1 Niche Definition Prompt
```
You are a senior SEO copywriter. Create a concise niche brief for a **{BUSINESS_TYPE}** located in **{CITY, STATE}**. Include:
- Core services (max 5)
- Primary local keywords (3‑5)
- Suggested page hierarchy (Home, Services, About, Contact, Blog)
- Schema types to implement (LocalBusiness, Service, Review, etc.)
Return JSON.
```
*Replace `{BUSINESS_TYPE}` and `{CITY, STATE}` with the target values.*

### 1.2 Home Page Copy Prompt
```
Write persuasive home‑page copy for a **{BUSINESS_TYPE}** using the keywords from the niche brief. Keep the tone friendly, include a clear CTA ("Call now"), and embed the primary keyword in the H1.
Output markdown.
```

### 1.3 Service Page Prompt
```
For each service listed in the niche brief, generate a 300‑word description optimized for the associated keyword. Include a short FAQ (2‑3 questions) and a bullet list of benefits.
Output markdown per service.
```

### 1.4 SEO Metadata Prompt
```
Create SEO title, meta description, and OpenGraph tags for each page generated above. Keep title ≤ 60 characters, description ≤ 160 characters. Include the primary keyword.
Return JSON mapping page → metadata.
```

### 1.5 Structured Data Prompt
```
Generate JSON‑LD schema for the business using the schema types from the niche brief. Populate name, address, phone, openingHours, and serviceOfferings. Output a single JSON block ready to embed in `<script type="application/ld+json">`.
```

---

## 2. Automation Scripts (provided)

- **generate_site.py** – Takes the JSON outputs from the prompts and builds a static site using Jinja2 templates.
- **seo_audit.py** – Runs Ahrefs/Google Search Console APIs to verify keyword rankings after deployment.
- **deploy.sh** – Deploys the Docker container to the leasing host and registers the site in the central database.

All scripts are located in the `scripts/` folder of the repository.

---

## 3. Tracking Incoming Enquiries

### 3.1 Telephone Call Recording
| Component | Recommended Service |
|-----------|---------------------|
| Virtual Phone Number | **Twilio Voice** (Programmable Voice) – $1 / month per number + per‑minute usage. |
| Call Recording & Storage | Twilio automatically records; store MP3s in an S3 bucket with lifecycle rules (30 days → Glacier). |
| Call Analytics | Use Twilio’s `transcriptions` and `duration` fields; push into a PostgreSQL `calls` table. |

### 3.2 Email Capture
- Set up a **domain‑wide email** using Google Workspace or Microsoft 365.
- Forward all inbound mail to a **Zapier/Make** workflow that creates a record in the `email_inquiries` table and optionally triggers a Slack notification.

### 3.3 Quote Request Form
- Embed a **HubSpot Free Form** or **Formspree** (no‑code) that posts to our `/api/inquiry` endpoint.
- The endpoint stores JSON payload (`name, email, phone, message, utm_source`) in the `form_requests` table.
- Use **SendGrid** or **Mailgun** to send an acknowledgement email with a unique ticket ID.

All three channels write to the same **central PostgreSQL database** (`enquiries` schema) so the leasing business can view a unified dashboard.

---

## 4. Dashboard Overview (for the leasing business)

| Tab | Data Shown |
|-----|------------|
| Calls | Date, duration, recording link, transcription snippet |
| Emails | Sender, subject, timestamp, reply status |
| Form Requests | Lead name, contact info, message, conversion funnel step |
| Analytics | Monthly traffic, keyword rankings, conversion rate |

The dashboard is built with **React + Tailwind** and queries the API using JWT authentication.

---

## 5. Quick Start Checklist
1. Choose a niche and run the **Niche Definition Prompt**.
2. Run the remaining prompts in order; save each JSON/markdown output.
3. Execute `python scripts/generate_site.py --data ./outputs`.
4. Commit the generated site to the Git repo and run `./scripts/deploy.sh`.
5. Set up Twilio number, email forwarding, and Formspree endpoint as described.
6. Verify inbound leads appear in the dashboard.

---

## 6. Recommended Tools & Services Summary
- **LLM**: Claude Fable via OpenRouter (API key).
- **Prompt orchestration**: Simple bash script or `invokeai` to call the OpenRouter API.
- **Static site generation**: Jinja2 + Python.
- **Hosting**: Docker on AWS Lightsail / DigitalOcean App Platform.
- **Phone**: Twilio Voice (recording).
- **Email**: Google Workspace (catch‑all) + Zapier.
- **Forms**: Formspree (free tier) or HubSpot Free.
- **Database**: PostgreSQL (managed).
- **Analytics**: Google Analytics 4, Search Console, Ahrefs API.
- **Dashboard**: React + Tailwind + Node/Express API.

---

## 7. Maintenance
- Monthly SEO audit (run `seo_audit.py`).
- Quarterly template refresh (update design assets).
- Weekly backup of recordings and database.

---

*End of Guide.*