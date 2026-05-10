# DeraBlog

> **A modern, fast, SEO-first content management system. Open source, self-hosted, and complete out of the box.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PHP Version](https://img.shields.io/badge/PHP-8.2%2B-777BB4.svg)](https://www.php.net/)
[![Laravel](https://img.shields.io/badge/Laravel-11-FF2D20.svg)](https://laravel.com/)
[![Status](https://img.shields.io/badge/status-pre--alpha-orange.svg)]()

---

## What DeraBlog Is

DeraBlog is a complete content publishing platform — built from the ground up — that includes everything a modern publisher, blogger, or agency needs to run a fast, secure, multilingual, search-optimized site, without bolting on dozens of separate add-ons.

Where most CMS platforms ship a minimal core and require paid extensions for essential capabilities, DeraBlog ships **all of the essentials in the Core, free, forever**.

---

## Core Capabilities

### Content & Editing
- Block-based editor with drag-and-drop authoring
- Custom content types and custom fields (15+ field types, including repeater, flexible content, relations, conditional logic)
- Unlimited revisions with side-by-side diff viewer
- Auto-save drafts, scheduled publishing, private posts
- Bulk operations across thousands of posts
- Content importer for migrating from existing platforms

### Search Engine Optimization
- Full meta tag management (title, description, OpenGraph, Twitter Cards)
- Comprehensive structured data (JSON-LD): Article, FAQ, Recipe, Product, How-to, Review, Event, Breadcrumb, Organization
- Real-time content analyzer with multiple focus keyword support
- Internal linking suggestions powered by content analysis
- Broken link detection and reporting
- Cornerstone content management
- XML sitemap generation, robots.txt management
- 301/302 redirects manager
- Schema templates library covering all common content types
- Optional Search Console integration

### Performance
- Page caching with intelligent invalidation
- Browser caching headers, CSS/JS minification and combination
- Critical CSS generation
- Automatic image optimization (WebP/AVIF)
- Lazy loading for images, videos, and iframes
- Database optimization tools
- Cache preloading (warm-up)
- CDN integration via URL rewriting
- Defer/async JavaScript loading

### Security
- Two-factor authentication (TOTP-based)
- Login attempt limiting and IP blocking
- Full activity log / audit trail
- File integrity monitoring
- Built-in malware scanner
- Web Application Firewall rules (lite)
- Brute force protection
- Hide-login-URL option
- Anti-spam (honeypot + rate limiting)

### Visual Page Builder
- Drag-and-drop visual builder with section/column system
- Responsive controls (desktop / tablet / mobile)
- Reusable blocks and saved templates
- Theme builder for custom headers and footers
- Global styles, custom CSS per block
- Live preview

### Multilingual
- Full RTL support (Arabic-first quality)
- Translate posts, pages, custom fields, custom post types, taxonomies
- Locale-prefixed URLs and automatic hreflang
- Local translation memory (no external services required)
- String translations for UI, themes, and plugins

### Editorial Calendar
- Drag-and-drop monthly/weekly/daily views
- Multi-author filtering, color coding by category
- Quick edit and bulk reschedule
- iCal export (subscribe from any calendar app)
- Content gap detection
- Recurring posts

### Automation Engine
- Visual node-based automation builder
- Triggers: post events, comments, subscribers, form submissions, schedules, webhooks, manual
- Actions: email, newsletter, social posting, tag/category updates, status changes, webhooks, AI actions, chat platform notifications
- Conditional logic, delays, loops, and filters
- Pre-built automation templates
- Test-run mode

### AI Features (Bring Your Own Key)
- Multi-provider support: **OpenAI, Deepseek, Gemini, OpenRouter**
- AI content writer, SEO suggestions, translator, summarizer
- Image generation, alt text generator
- Tag auto-suggestion, schema generator, outline generator
- Content repurposing, comment moderation
- Internal linking suggestions
- Cost tracking dashboard with monthly limits
- The user supplies their own API key — features are free; the user pays only for their own provider usage

### Forms
- Drag-and-drop form builder
- Multi-step forms, conditional logic, file uploads
- Email notifications, confirmation pages
- Submissions database with CSV export
- Pre-built templates (contact, survey, registration, quote)

### Backup & Migration
- Manual and scheduled backups (database + files)
- Local storage and remote storage (S3-compatible, BYOK)
- Encrypted backups
- One-click restore
- Cross-server migration tool

### Newsletter
- Built-in subscriber management
- Send via SMTP (no third-party service required)
- Email template editor
- Tags and segments
- RSS-to-email auto-send
- Double opt-in, unsubscribe management
- Privacy-respecting open and click tracking

### Privacy-First Analytics
- Page views, top posts, top referrers, search queries
- Real-time visitor count
- Geographic stats (local IP database)
- Browser and device breakdown
- Outbound link tracking, 404 error log
- No external trackers, no GDPR cookie banner required

### Membership
- Subscriber-only content and member-only categories
- Free membership tiers
- Drip content (release over time)
- Members directory
- Optional payment integration (BYOK)

### Comments
- Threaded comments with reactions
- Edit and delete (within configurable window)
- Moderation queue, anti-spam
- Email notifications, Markdown support
- Voting (up/down)

### Themes & Plugins
- Theme system with `theme.json` manifests, slot system, dark mode, RTL
- Plugin system with simple service-provider registration
- Hooks system (Actions and Filters)
- Plugin/theme installer with one-click activation
- Auto-update mechanism for licensed add-ons
- Plugin marketplace browser (read-only at launch)

### Developer Tools
- REST API (token-authenticated)
- Webhooks
- CLI tool for plugin/theme scaffolding
- Comprehensive developer documentation

---

## Stack

- **Backend:** Laravel 11 + Filament 3
- **Frontend:** Livewire 3 + Alpine.js + Blade + Tailwind CSS (TALL Stack)
- **Database:** MySQL 8
- **Cache / Queue / Sessions:** Redis
- **Real-time:** Laravel Reverb (WebSockets)
- **Editor:** TipTap (block editor) with Alpine wrapper
- **Calendar UI:** FullCalendar
- **Automation Builder:** Drawflow

---

## Installation

> **Status:** Pre-alpha. Not ready for production. Soft Launch is targeted for Week 12 of development.

### Docker (recommended)

```bash
git clone https://github.com/derabia/DeraBlog.git
cd DeraBlog
cp .env.example .env
docker compose up -d
docker compose exec app php artisan migrate --seed
```

Visit `http://localhost` for the public site, `http://localhost/admin` for the admin panel.

### One-line install (production VPS)

```bash
curl -fsSL https://get.derablog.com/install | bash
```

*(Available after the v1.0 release.)*

---

## Documentation

- [PLAN.md](PLAN.md) — Master development plan
- [ARCHITECTURE.md](ARCHITECTURE.md) — Technical architecture
- [BUSINESS.md](BUSINESS.md) — Business strategy
- [CONTRIBUTING.md](CONTRIBUTING.md) — How to contribute
- [SECURITY.md](SECURITY.md) — Security policy & vulnerability disclosure
- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) — Community guidelines

---

## License

DeraBlog Core is released under the **MIT License** — see [LICENSE](LICENSE).

Premium themes and add-ons sold separately at [derablog.com](https://derablog.com) are released under a **Commercial EULA** — see [LICENSE-COMMERCIAL.md](LICENSE-COMMERCIAL.md).

---

## Roadmap

- **Weeks 1–6:** Core foundation (content management, authentication, themes, plugins, baseline SEO and caching)
- **Weeks 7–12:** Full SEO suite, performance suite, security suite, forms, backup, newsletter, analytics, multilingual → 🚀 **Soft Launch**
- **Weeks 13–18:** Page builder, theme builder, membership, editorial calendar
- **Weeks 19–24:** Automation engine, AI features → 🚀 **Full Launch v1.0**

See [PLAN.md](PLAN.md) for the detailed roadmap.

---

## Status

DeraBlog is being built solo by [@Amrwebdeveloper](https://github.com/Amrwebdeveloper), the founder of [Derabia.com](https://derabia.com). Follow along as we build in public. The repository is **pre-alpha**; v1.0 is targeted for Week 24 of development.

⭐ Stars and watches welcome.

---

## Acknowledgments

DeraBlog is built on the shoulders of:
- [Laravel](https://laravel.com/) — the PHP framework
- [Livewire](https://livewire.laravel.com/) — full-stack reactivity
- [Filament](https://filamentphp.com/) — admin panel toolkit
- [Tailwind CSS](https://tailwindcss.com/) — utility-first styling
- [Alpine.js](https://alpinejs.dev/) — minimal client-side reactivity
- [TipTap](https://tiptap.dev/) — block editor framework
- [FullCalendar](https://fullcalendar.io/) — calendar UI
- [Drawflow](https://github.com/jerosoler/Drawflow) — node-based UI library
