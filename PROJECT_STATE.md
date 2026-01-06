# VibeMonies - Project State & Development Guide

**Last Updated:** January 6, 2026
**Live Site:** https://vibemonies.com
**GitHub Repo:** https://github.com/rijad-krasniqi/vibemonies

---

## Project Overview

VibeMonies is a Hugo-based blog focused on three main content areas:
- **Prediction Markets** - Trading, platforms, strategies
- **Vibe Coding** - AI-assisted development, tools, workflows
- **Making Money** - Digital economy, side hustles, monetization

---

## Current Status

### Content (49 Articles Total)
| Category | Count | Location |
|----------|-------|----------|
| Prediction Markets | 17 | `content/prediction-markets/` |
| Vibe Coding | 17 | `content/vibe-coding/` |
| Making Money | 15 | `content/making-money/` |

### Deployment
- **Hosting:** Netlify
- **Domain:** vibemonies.com (SSL enabled via Netlify DNS)
- **CMS:** Decap CMS at `/admin/` (uses Netlify Identity + Git Gateway)
- **Build Command:** `hugo`
- **Publish Directory:** `public`

---

## Implemented Features

### SEO Optimizations
- [x] JSON-LD Article structured data (author, publisher, wordCount, articleSection)
- [x] BreadcrumbList schema for Google rich snippets
- [x] Meta descriptions and keywords on all pages
- [x] Canonical URLs
- [x] Open Graph tags for social sharing
- [x] Twitter Card meta tags
- [x] Sitemap auto-generated at `/sitemap.xml`
- [x] "Last Updated" dates shown when modified

### User Experience
- [x] Reading progress bar (top of page on articles)
- [x] Collapsible Table of Contents
- [x] Breadcrumb navigation
- [x] Social sharing buttons (Twitter/X, LinkedIn, Facebook, Copy Link)
- [x] Author bio section on articles
- [x] Copy code buttons on code blocks
- [x] Lazy loading images
- [x] Drop cap first letter styling
- [x] Related posts section

### Analytics
- [x] Google Analytics 4: `G-9JPX80EJR4`
- [ ] Google Search Console (manual setup required - submit sitemap)

---

## Key Files

### Templates
```
layouts/
├── _default/
│   ├── baseof.html      # Base template, contains GA, all JS
│   ├── single.html      # Article page (SEO schema, TOC, sharing)
│   ├── list.html        # Category/tag listing pages
│   └── terms.html       # Taxonomy term pages
├── index.html           # Homepage
└── partials/
    ├── header.html      # Site header/navigation
    └── footer.html      # Site footer
```

### Styles
```
static/css/style.css     # All CSS (~1000 lines)
```

### CMS
```
static/admin/
├── index.html           # CMS entry point
└── config.yml           # Decap CMS configuration
```

### Configuration
```
hugo.toml                # Hugo site configuration
```

---

## Local Development

### Prerequisites
- Hugo Extended v0.110+ (currently using v0.154.2)
- Git

### Commands
```bash
# Navigate to project
cd vibemonies-hugo

# Start development server
hugo server -D

# Build for production
hugo

# View site at http://localhost:1313
```

---

## CMS Access

1. Go to https://vibemonies.com/admin/
2. Login with Netlify Identity
3. Create/edit posts in any category
4. Changes auto-commit to GitHub and trigger Netlify rebuild

### CMS Backend
- Uses `git-gateway` with Netlify Identity
- Config at `static/admin/config.yml`
- Media uploads go to `static/images/uploads/`

---

## Pending/Future Tasks

### Immediate (User Working On)
- [ ] Add featured images to all articles
- [ ] Edit and polish article content

### Recommended Next Steps
- [ ] Submit sitemap to Google Search Console
- [ ] Set up Google Search Console property verification
- [ ] Create custom 404 page
- [ ] Add newsletter signup integration (e.g., ConvertKit, Mailchimp)
- [ ] Add comments system (e.g., Giscus, Disqus)
- [ ] Create an About page
- [ ] Add RSS feed link in footer

### Nice to Have
- [ ] Dark mode toggle
- [ ] Search functionality
- [ ] Reading list/bookmarks feature
- [ ] Performance audit (Lighthouse)

---

## Quick Reference

### Add New Article via CLI
```bash
hugo new prediction-markets/my-new-article.md
hugo new vibe-coding/my-new-article.md
hugo new making-money/my-new-article.md
```

### Deploy Changes
```bash
git add .
git commit -m "Description of changes"
git push origin main
# Netlify auto-deploys on push
```

### View Analytics
- Google Analytics: https://analytics.google.com
- Property: G-9JPX80EJR4

### Submit Sitemap
1. Go to https://search.google.com/search-console
2. Add property: vibemonies.com
3. Verify ownership (DNS or HTML file)
4. Submit sitemap: https://vibemonies.com/sitemap.xml

---

## Troubleshooting

### CMS Login Issues
- Ensure Netlify Identity is enabled in site settings
- Enable Git Gateway in Identity > Services
- Check invited users in Identity tab

### Build Failures
- Check Hugo version compatibility
- Run `hugo --verbose` locally to debug

### Template Errors
- Nil pointer errors often mean missing `.File` on taxonomy pages
- Use `{{ with }}` blocks to safely access nested properties

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| Static Site Generator | Hugo Extended |
| Hosting | Netlify |
| CMS | Decap CMS (formerly Netlify CMS) |
| Authentication | Netlify Identity |
| Analytics | Google Analytics 4 |
| Fonts | Space Grotesk, DM Sans, JetBrains Mono |
| Version Control | Git + GitHub |

---

## Contact

Repository Owner: rijad-krasniqi
Site: https://vibemonies.com

---

*This document serves as a handoff guide for continuing development across devices or sessions.*
