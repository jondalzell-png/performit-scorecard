# AI Character Readiness Scorecard — Technical Documentation

## Overview

The AI Character Readiness Scorecard is a lead generation tool for Performit Live. It assesses how prepared an organisation is to deploy AI digital characters by evaluating four key dimensions. The tool captures leads, scores them, delivers a personalised PDF report, and feeds the data into Certainty Engine CRM for automated follow-up.

**Live URL:** https://performitlive.com/scorecard

---

## Tech Stack

| Layer | Technology | Details |
|-------|-----------|---------|
| Frontend | Vanilla HTML/CSS/JS | Single-file application (`index.html`, ~170KB) |
| Fonts | Google Fonts | Inter (weights 300–900) via CDN |
| CRM | Certainty Engine | GoHighLevel-based, Inbound Webhook integration |
| Email | Certainty Engine | Branded HTML email triggered by workflow |
| Hosting | SiteGround | Static files in `public_html/scorecard/` |
| Source Control | GitHub | `jondalzell-png/performit-scorecard` (main branch) |
| PDF Generation | Browser Print | `window.open()` + `window.print()` — no server-side dependencies |
| Booking | Performit Live Widget | `links.performitlive.com/widget/booking/7yI8QxA6rWZxjB6bxbS8` |

### No External JavaScript Dependencies

The entire application runs on vanilla JavaScript — no React, jQuery, or other frameworks. This means zero dependency management, no build step, and no package vulnerabilities to maintain.

---

## Architecture

```
User visits performitlive.com/scorecard
        │
        ▼
   Landing Page (index.html)
   ├── Hero + lead capture form (First Name, Last Name, Email, Organisation)
   ├── Social proof section (client logos)
   ├── "What You'll Learn" section
   └── "How It Works" section
        │
        ▼ (user submits form)
   Quiz (20 questions, 4 categories)
   ├── Deployment Clarity (5 questions, max 500pts)
   ├── Expectations & Behaviour (5 questions, max 500pts)
   ├── Ownership & Accountability (5 questions, max 500pts)
   └── Response & Oversight (5 questions, max 500pts)
        │
        ▼ (quiz complete)
   Loading Sequence (animated)
        │
        ├──▶ submitToCertaintyEngine() ──▶ Webhook POST
        │                                    │
        │                              Certainty Engine Workflow:
        │                              1. Create/Update Contact
        │                              2. Set Custom Fields (scores)
        │                              3. Add Tag: scorecard-complete
        │                              4. Send Branded HTML Email
        │
        ▼
   Results Page
   ├── Overall score gauge (colour-graded red→amber→green)
   ├── Tier badge (Foundation Stage / Developing Readiness / Operational Readiness)
   ├── 4 category score cards with descriptions
   ├── Priority gap analysis (lowest-scoring area)
   ├── CTA: Book a Strategy Call
   └── Download Your Report (PDF)
        │
        ▼
   9-Page PDF Report (generated in browser)
   ├── Cover page
   ├── Your Overall Score
   ├── Scores at a Glance (category breakdown)
   ├── Gap analysis (lowest-scoring area)
   ├── Building Readiness Framework (2 pages)
   ├── Pilot Readiness Checklist
   ├── Operational Readiness Scorecard (ongoing)
   ├── Where to Start (next steps)
   └── CTA page with hero image
```

---

## Scoring System

### Tiers

| Tier | Score Range | Description |
|------|-----------|-------------|
| Foundation Stage | 0–35% | Foundational conditions not yet in place |
| Developing Readiness | 36–74% | Structures forming, gaps remain |
| Operational Readiness | 75–100% | Strong foundations across all areas |

### Developing Readiness Sub-bands

| Sub-band | Score Range |
|----------|-----------|
| Lower Developing | 36–50% |
| Mid Developing | 51–60% |
| Upper Developing | 61–74% |

### Category Scoring

Each category has 5 questions. Each question has 4 answer options with ascending scores. All categories have a maximum of 500 points. Overall score is calculated as: `total points / 2000 * 100`.

### Colour Interpolation

Scores are colour-graded dynamically:
- **0%** = Red (RGB 255, 70, 70)
- **50%** = Amber (RGB 255, 155, 0)
- **100%** = Green (RGB 94, 219, 94)

All UI elements (gauge, cards, badges, PDF) use this same colour system.

---

## Certainty Engine Integration

### Webhook

- **URL:** `https://services.leadconnectorhq.com/hooks/g1m8j2MpXVdfvFK2gJWj/webhook-trigger/788dda06-0d21-4a7e-811c-76d4f1915be3`
- **Method:** POST
- **Content-Type:** application/json
- **Fires:** When user completes the quiz (before results are shown)

### Payload

```json
{
  "firstName": "Jon",
  "lastName": "Dalzell",
  "email": "jon@example.com",
  "industry": "Retail",
  "role": "C-Suite / Executive",
  "overallScore": "69",
  "tier": "Developing Readiness",
  "deploymentClarityScore": "50",
  "expectationsBehaviourScore": "48",
  "ownershipAccountabilityScore": "52",
  "responseOversightScore": "52",
  "reportUrl": "https://performitlive.com/scorecard/?r=eyJ..."
}
```

### Workflow (Certainty Engine)

**Name:** Scorecard Results Email

1. **Trigger:** Inbound Webhook
2. **Create Contact** — maps firstName, lastName, email from webhook
3. **Update Contact Fields** — maps all score fields:
   - `contact.scorecard_overall_score`
   - `contact.scorecard_tier`
   - `contact.deployment_clarity_score`
   - `contact.expectations_behaviour_score`
   - `contact.ownership_accountability_score`
   - `contact.response_oversight_score`
   - `contact.scorecard_report_url`
4. **Add Tag** — `scorecard-complete`
5. **Send Email** — branded HTML email with scores, two CTAs, social icons

### Custom Fields in CRM

| Field Key | Type |
|-----------|------|
| contact.scorecard_overall_score | Single Line |
| contact.scorecard_tier | Single Line |
| contact.deployment_clarity_score | Single Line |
| contact.expectations_behaviour_score | Single Line |
| contact.ownership_accountability_score | Single Line |
| contact.response_oversight_score | Single Line |
| contact.scorecard_report_url | Single Line |
| contact.scorecard_completed_date | Single Line |

---

## Email Template

The results email uses a **custom HTML template** (Code Editor in Certainty Engine Smart Builder):

- Dark theme (#0a0a0a background) matching the scorecard brand
- Overall score box with tier badge
- 4 category score rows
- **"View Your Full Report"** button (orange, solid) — links to report URL
- **"Book a Free Strategy Call"** button (orange, outlined) — links to booking widget
- Instagram and LinkedIn social icons in footer
- From: Performit Live (info@mail.performitlive.com)

---

## Report Link System

When the quiz is submitted, a shareable report URL is generated:

```
https://performitlive.com/scorecard/?r=eyJuIjoiSm9uIERhbHplbGwi...
```

The `?r=` parameter contains a Base64-encoded JSON object:

```json
{
  "n": "Jon Dalzell",      // name
  "e": "jon@example.com",  // email
  "i": "Retail",           // industry
  "ro": "C-Suite",         // role
  "s": [50, 48, 52, 52]    // category percentages
}
```

When a user clicks this link (e.g. from the email), the page auto-renders their results without retaking the quiz.

---

## File Structure

```
Landing Page/
├── index.html              # Main file (landing page + quiz + results + PDF report)
├── quiz.html               # Legacy file (older version, not actively used)
├── performit-logo.png      # Main logo (used in nav, PDF cover)
├── performit-logo-black.png # Dark logo variant
├── performitlogo.png       # Logo duplicate
├── hero-kiosk.png          # CTA section hero image (also used in PDF page 9)
├── hero-kiosk.jpg.png      # Hero image variant
├── report-mockup.png       # Report mockup image (landing page)
├── frameless-logo.png      # Client logo — Frameless
├── logo-wonderlosity.webp  # Client logo — Wonderlosity
├── logo-sky.webp.png       # Client logo — Sky
├── universal-logo.png      # Client logo — Universal
├── ocean-logo.png          # Client logo — Ocean Outdoor
├── sky-logo.png            # Client logo — Sky (duplicate)
├── Goodwood-Festival-of-Speed.webp  # Client logo
├── Waterloo-Uncovered-logo.webp     # Client logo
├── netlify.toml            # Netlify config (legacy, not used)
└── netlify/functions/      # Netlify proxy function (legacy, not used)
```

---

## Hosting & Deployment

### Current Setup

- **Host:** SiteGround (shared hosting with WordPress)
- **Path:** `public_html/scorecard/`
- **WordPress Bypass:** `.htaccess` rule added to prevent WordPress intercepting `/scorecard` URLs:
  ```
  RewriteRule ^scorecard(/.*)?$ - [L]
  ```

### Deployment Process

1. Make changes to `index.html` locally (at `C:\Users\jonda\Desktop\Landing Page\`)
2. Commit and push to GitHub: `git add index.html && git commit -m "message" && git push`
3. Upload updated `index.html` to SiteGround via File Manager (`public_html/scorecard/`)

**Important:** Step 3 is manual — there is no CI/CD pipeline. The GitHub repo is for version control only. The live site is served from SiteGround, not GitHub.

---

## Maintenance

### Routine Tasks

| Task | Frequency | Details |
|------|-----------|---------|
| Check email delivery | Monthly | Verify emails are landing in inbox, not spam |
| Review CRM contacts | Weekly | Check new scorecard-complete contacts in Certainty Engine |
| Update copyright year | Annually | Footer of index.html |
| Check booking link | Monthly | Ensure booking URL is still active |
| Monitor SiteGround | Monthly | Check hosting, SSL, uptime |

### Changing Questions

Questions are defined in the `QS` array in index.html (around line 2680). Each question has:

```javascript
{c: 0, t: 'Question text', a: [
  {t: 'Answer A', s: 25},
  {t: 'Answer B', s: 50},
  {t: 'Answer C', s: 75},
  {t: 'Answer D', s: 100}
]}
```

- `c` = category index (0–3)
- `t` = question/answer text
- `s` = score value for that answer

### Changing Tiers

Tiers are defined in the `TIERS` array (around line 2731). Each tier has `label`, `min`, `max`, heading text, summary text, and bullet points.

### Changing the PDF Report

The PDF is generated by `buildReportHTML()` (around line 3059). It returns a complete HTML document string. Editing requires HTML/CSS knowledge. The report uses the same colour interpolation as the on-screen results.

---

## Cloning This Approach for Other Scorecards

### What You Can Reuse (the Template)

The scorecard architecture is designed around a repeatable pattern. To create a new scorecard for a different topic, you can reuse:

1. **The full page structure** — landing page → quiz → results → PDF → CRM integration
2. **The scoring engine** — category-based scoring with configurable tiers and colour grading
3. **The PDF report generator** — 9-page branded report structure
4. **The CRM integration pattern** — webhook → create contact → set fields → tag → email
5. **The email template** — branded HTML email with score summary and CTAs
6. **The report link system** — Base64-encoded URL params for shareable results

### Step-by-Step Cloning Process

#### 1. Copy the Template

```bash
cp -r "Landing Page" "New Scorecard"
```

#### 2. Define Your Scorecard Content

Create a content brief with:

| Element | What to Define |
|---------|---------------|
| **Topic** | What the scorecard assesses (e.g., "Digital Maturity", "Brand Safety") |
| **Categories** | 3–5 scoring dimensions |
| **Questions** | 4–6 questions per category, each with 4 graded answer options |
| **Tiers** | 2–4 result tiers with names, descriptions, and score ranges |
| **Gap Analysis** | Per-category analysis text for each tier |
| **Next Steps** | Tier-specific recommended actions |
| **PDF Content** | Building readiness framework, checklists, operational scorecard |

#### 3. Update the Code

**a) Branding (CSS variables at top of file):**
```css
:root {
  --orange: #FF9B00;      /* Change to your brand colour */
  --orange-dark: #D98300;  /* Darker variant */
  --black: #0A0A0A;        /* Background */
}
```

**b) Landing Page Content:**
- Update hero headline, subheadline, bullet points
- Replace client logos with relevant ones
- Update "What You'll Learn" and "How It Works" sections

**c) Quiz Data (JavaScript arrays):**

```javascript
// Categories
const CATS = [
  {id:'cat1', name:'Category Name', max:500, desc:'Description...'},
  // ... more categories
];

// Questions
const QS = [
  {c:0, t:'Question text?', a:[
    {t:'Low maturity answer', s:25},
    {t:'Below average answer', s:50},
    {t:'Above average answer', s:75},
    {t:'High maturity answer', s:100}
  ]},
  // ... more questions
];

// Tiers
const TIERS = [
  {label:'Tier Name', cls:'t-nr', min:0, max:35,
   h:'Heading for this tier',
   sub:'Subheading',
   s:'Summary paragraph',
   b:['Bullet 1','Bullet 2','Bullet 3']},
  // ... more tiers
];
```

**d) Gap Analysis Data:**
Update the `GAP` object and `CAT_DESC` object with category-specific analysis text.

**e) PDF Report:**
Update `buildReportHTML()` with new content for:
- Cover page title
- Framework guidance pages
- Checklist items
- Operational scorecard items

#### 4. Set Up CRM Integration

In Certainty Engine (or any GoHighLevel-based CRM):

1. **Create custom fields** for the new scorecard's scores
2. **Create a new workflow** with Inbound Webhook trigger
3. **Send a test POST** to generate the mapping reference
4. **Map the fields** in each workflow action
5. **Create the email template** (clone the HTML and update content)
6. **Update the webhook URL** in the code's `submitToCertaintyEngine()` function

#### 5. Deploy

1. Create a new SiteGround subfolder (e.g., `public_html/new-scorecard/`)
2. Add `.htaccess` bypass rule: `RewriteRule ^new-scorecard(/.*)?$ - [L]`
3. Upload all files
4. Test the full flow end-to-end

### What Changes Per Scorecard

| Component | Effort | Notes |
|-----------|--------|-------|
| Landing page copy | Low | Update text, images, headlines |
| Questions & answers | Medium | Core content — requires subject matter expertise |
| Tier definitions | Medium | Names, descriptions, score ranges |
| Gap analysis text | Medium | Per-category, per-tier analysis and recommendations |
| PDF report content | High | Framework pages, checklists, operational scorecard |
| CRM setup | Low | New fields, new workflow, new webhook URL |
| Email template | Low | Clone and update copy |
| Branding/colours | Low | CSS variables |

### What Stays the Same

- Page structure and layout
- Quiz engine and scoring logic
- Colour interpolation system
- PDF generation framework
- CRM webhook integration pattern
- Report link system (Base64 URL params)
- Email template structure
- Responsive design and animations

### Future Improvement: Extract a Reusable Template

To make cloning even easier, consider extracting the scorecard into a configuration-driven template:

1. **Separate content from code** — move all questions, tiers, gap text, and PDF content into a JSON config file
2. **Create a template `index.html`** that reads from the config
3. **New scorecard = new JSON file** — no code changes needed, just content

This would reduce the effort per new scorecard from hours to minutes, requiring only content creation rather than code editing.

---

## Key Contacts & Resources

| Resource | Location |
|----------|----------|
| GitHub Repo | github.com/jondalzell-png/performit-scorecard |
| SiteGround | tools.siteground.com (performitlive.com) |
| Certainty Engine | app.certaintyengine.io |
| Booking Widget | links.performitlive.com/widget/booking/7yI8QxA6rWZxjB6bxbS8 |
| Instagram | instagram.com/performit_live |
| LinkedIn | linkedin.com/company/performit-live |
| Sending Email | info@mail.performitlive.com |
