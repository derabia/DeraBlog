# DeraBlog — Master Plan

> **Owner:** Derabia.com
> **Product:** DeraBlog — A modern, complete, value-first content management system on the Laravel TALL Stack
> **License:** Hybrid — MIT (Core CMS) + Commercial EULA (Premium Themes only)
> **Status:** Pre-development planning phase
> **Last updated:** 2026-05-10
> **Companion docs:** [`ARCHITECTURE.md`](ARCHITECTURE.md) (technical), [`BUSINESS.md`](BUSINESS.md) (commercial), [`SECURITY-DESIGN.md`](SECURITY-DESIGN.md) (security architecture, modules, gates)

---

## 1. Vision

Build a fast, SEO-first, **truly complete** open-source content management system that includes — out of the box — every capability a modern publisher, blogger, or agency needs to run a professional site.

**Core Philosophy:** Adoption first, monetization later. The competitive moat is **value delivered**, not features withheld.

**Two audiences:**
1. **Personal use** — Derabia.com's own publication.
2. **Commercial product** — bloggers, publishers, agencies adopting DeraBlog as their primary CMS.

**The Switching Argument:**
> *"DeraBlog includes the complete toolkit out of the box: full SEO suite, performance optimization, security hardening, multilingual support, page builder, automation engine, AI integrations, editorial calendar, membership, newsletter, analytics, backups, and forms — all in the Core, no add-ons required, no recurring fees."*

---

## 2. Locked Decisions

| Decision | Choice | Why |
|---|---|---|
| Stack | **TALL — Tailwind + Alpine + Laravel + Livewire** | Solo-dev velocity, single language (PHP), runs on shared hosting |
| Backend | Laravel 11 + Filament 3 | Mature ecosystem; Filament is built on Livewire (paradigm consistency) |
| Database | MySQL 8 | Universal hosting support |
| Cache / Queue / Session | Redis | Standard, fast |
| WebSocket | Laravel Reverb | Native Laravel, easy install |
| Editor | TipTap (Alpine wrapper) | Modern block editor |
| Calendar | FullCalendar | Open source, vanilla JS, integrates cleanly with Alpine |
| Automation Flow | Drawflow | Vanilla JS, TALL-compatible (xyflow is React-only) |
| AI Integration | **BYOK** (Bring Your Own Key) | Free in Core; user pays only for their own API usage |
| AI Providers | **OpenAI + Deepseek + Gemini + OpenRouter** | Coverage of major providers + cheapest (Deepseek) + multi-model router |
| License Model | **Hybrid: MIT (Core) + Commercial (Premium themes)** | Open Core for adoption; themes are designed assets |
| Code Protection | License Server + EULA, **NO encryption** | License-key validation + domain locking; encoder loaders unnecessary |
| Distribution | Self-hosted (VPS, shared hosting, Docker) | Customer owns their infrastructure |
| Payments | Lemon Squeezy | Handles VAT, license keys, file delivery globally |
| i18n | spatie/laravel-translatable + Laravel built-in | Default English, full RTL support for Arabic |
| Containerization | Docker Compose (production), hybrid (dev) | One-command install for customers |
| Repo Strategy | Single public Laravel repo (Core) + private repos for premium themes | Clear open / commercial separation |
| Timeline | **Option B — Soft Launch at Week 12, Full Launch at Week 24** | Get user feedback early, polish before full launch |
| **Security architecture** | **5 cross-cutting modules under `app/Security/`: `EgressGuard`, `AIGateway`, `ArtifactTrust`, `ContentSanitizer`, `AuthEdge`** | Each module is a single chokepoint that closes a cluster of audit findings; details in `SECURITY-DESIGN.md` |
| **Plugin / theme trust** | **Two-tier: Verified (Minisign-signed by Derabia.com) and Community (unsigned, requires per-install admin consent, no auto-update)** | Forced binary choice would either kill community or kill brand |
| **AI cost ceiling** | **Hard $50/mo default per install, env-overridable up to $500, NOT admin-overridable** | Reputational ATO risk on "DeraBlog drained my key" outweighs admin convenience |
| **Compliance scope** | **GDPR-ready by design + Egypt 151/2020 (operator residency); PDPL-SA acknowledged** | GDPR is the strictest declared scope; satisfying it satisfies the rest |
| **CSP** | **Strict-CSP with per-request nonce, `strict-dynamic`, no `unsafe-inline`** | Only XSS posture that survives Page Builder + plugins + themes |
| **Telemetry** | **OFF by default; opt-in during onboarding; aggregate metrics only** | Privacy-by-default is the brand promise |

---

## 3. The Four Pillars

### Pillar 1: Hooks System (Actions + Filters)
- `DeraHooks::addAction('post.published', $callback)` — run code on events.
- `DeraHooks::addFilter('post.title', $callback)` — modify values before output.
- Pure PHP — single paradigm, no frontend mirror needed.

### Pillar 2: Custom Post Types + Custom Taxonomies
- Admin can define content types without code (Books, Recipes, Products, etc.).
- Each type has its own fields and taxonomies.

### Pillar 3: Custom Fields (Field Builder)
- 15+ field types: text, textarea, richtext, number, image, file, select, repeater, relation, flexible content, clone, conditional logic.
- Stored as JSON column on entries.
- Local JSON storage option for version control.

### Pillar 4: Themes + Plugins
- **Themes:** directory with `theme.json` + Blade templates + assets + Tailwind config.
- **Plugins:** directory with `plugin.json` + Service Provider (PHP) + optional Livewire components.

---

## 4. Core Feature Areas (~200 features across 19 areas)

The Core ships complete. Every capability listed below is in the Core CMS, free, with no extension required.

### 4.1 Content & Editing (15)
Posts/Pages CRUD, Categories & Tags, Custom Post Types, Custom Fields (15 types), Block Editor (TipTap), Media Library, Draft/Published/Scheduled/Private statuses, Auto-save drafts, **Unlimited Revisions with diff viewer**, Bulk operations, Import/Export (JSON), Content Importer (for migration from other platforms), Live Preview, Featured Images, Excerpts.

### 4.2 Authentication & Users (10)
Registration, Email Verification, Password Reset, **2FA (TOTP)**, 6 Default Roles, Custom Roles, Granular Permissions (40+), User Profiles, Strong Password Enforcement, Session Management.

### 4.3 SEO Suite (15)
Meta tags (title, description, OpenGraph, Twitter Cards), XML Sitemap, robots.txt management, 301/302 Redirects Manager, Full JSON-LD (Article, FAQ, Recipe, Product, How-to, Review, Event, Organization, BreadcrumbList), Real-time Content Analyzer, Schema Templates Library, Internal Linking Suggestions, Broken Link Checker, Multiple Focus Keywords, Cornerstone Content, Bulk SEO Editor, Image SEO Checker, Content Pillars, Search Console Integration (BYOK — user provides Google OAuth).

### 4.4 Performance Suite (12)
Page Caching, Browser Caching Headers, CSS/JS Minification, CSS/JS Combine, Critical CSS Generation, **Image Optimization (WebP/AVIF)**, Lazy Loading, Database Cleanup, Cache Preloading, CDN Integration, Defer/Async JS, Heartbeat Control.

### 4.5 Security Suite (10)
Login Attempt Limiting, IP Blocking, **Activity Log (full audit)**, File Integrity Monitoring, Malware Scanner, Firewall Rules (WAF-lite), Auto-logout, Hide Login URL, Brute Force Protection, Honeypot Anti-spam.

### 4.6 Forms System (10)
Drag-Drop Form Builder, Email Notifications, Multi-step Forms, File Uploads, Conditional Logic, Submissions Database, Export to CSV, Confirmation Pages, Spam Protection, Pre-built Templates.

### 4.7 Backup System (8)
Manual Backups, Scheduled Backups, DB + Files Backup, Local Storage, **S3-compatible (BYOK)**, One-click Restore, Backup Encryption, Migration Tool.

### 4.8 Multilingual Suite (11)
Multi-language UI, **Full RTL Support (Arabic-first quality)**, Translate all content types, Translate Custom Fields, Translate Taxonomies, URL Prefix per Locale, Hreflang automatic, Translation Memory (local), Translation Status Dashboard, String Translations, Auto-redirect by browser locale.

### 4.9 Page Builder (10)
**Drag-Drop Visual Builder**, Pre-made Sections Library, Responsive Controls (desktop/tablet/mobile), Reusable Blocks, Section/Column System, Save Templates, **Theme Builder (custom header/footer)**, Global Styles, Custom CSS per Block, Live Preview.

### 4.10 Comments System (10)
Built-in Comments, **Threading**, **Reactions (emoji)**, Edit/Delete (within window), Anti-spam, Moderation Queue, Email Notifications, Markdown Support, Voting (up/down), Avatar Support.

### 4.11 Newsletter System (10)
Subscribe Forms, Subscriber Management, **Send via SMTP** (no third-party service required), Email Templates Editor, Tags/Segments, **RSS-to-email auto-send**, Double Opt-in, Unsubscribe Management, Send Logs, Open/Click Tracking (privacy-first).

### 4.12 Privacy-First Analytics (12)
Page Views per Post, Top Posts, Top Referrers, Search Queries Log, **No GDPR cookie banner needed**, **Real-time Visitors**, Geographic Stats (local IP DB), Browser/Device Stats, Outbound Link Tracking, 404 Errors Log, Author Stats, Category/Tag Performance.

### 4.13 Membership Basic (8)
Subscriber-only Content, Member-only Categories, Free Membership Tiers, **Drip Content**, Members Directory, **Payment Integration (BYOK)**, Welcome Emails, Member Profile Pages.

### 4.14 Editorial Calendar (10)
**Drag-Drop Calendar View** (FullCalendar), Month/Week/Day Views, Multi-Author Filter, Status Filter, Color Coding, Quick Edit, Bulk Reschedule, iCal Export, Content Gap Detection, Recurring Posts.

### 4.15 Automation Engine (15)
**Visual Node Editor (Drawflow)**, Triggers (Post events, Comment, Subscriber, Form, User, Schedule, Webhook, Manual), Actions (Email, Newsletter, Social Post, Tag, Status Update, Notification, Webhook, AI Action, Chat platform notifications), If/Else Conditions, Wait/Delay, Loop, Filter, Pre-made Templates, Test Run Mode.

### 4.16 AI Features (BYOK) (12)
Multi-provider support (**OpenAI**, **Deepseek**, **Gemini**, **OpenRouter**) powers 12 AI capabilities: AI Content Writer, AI SEO Suggestions, AI Translation, AI Summarizer, AI Image Generation, AI Alt Text Generator, AI Tags Auto-suggest, AI Content Repurposing, AI Comment Moderation, AI Internal Linking, AI Schema Generator, AI Outline Generator.

### 4.17 Themes Infrastructure (8)
Default Theme (gorgeous), Theme Installer, Theme Settings UI, Dark Mode, RTL Support, Theme Hierarchy, Slot System, Live Customizer.

### 4.18 Plugins Infrastructure (7)
Plugin Loader, Plugin Activate/Deactivate, Plugin Install from ZIP, **Hooks System (Actions + Filters)**, Plugin Auto-update, Plugin Permissions, Plugin Marketplace UI (read-only browser).

### 4.19 Developer Tools (5)
REST API (Sanctum), Webhooks, CLI Tool (`php artisan dera:*`), Plugin Generator, Theme Generator.

---

## 5. Premium Strategy (Limited)

In Year 1, the **only** revenue stream is Premium Themes. This is intentional:
- Adoption is the priority.
- The free Core stands on its own as a complete CMS.
- Themes are a separate value (design/visual work) that justifies cost.

| Stream | Pricing | Launch |
|---|---|---|
| **Premium Themes** (designed) | $49–$99 | Month 4+ |
| **Custom Development** | $80–$150/hr | On request |
| **Priority Support** | $99–$499/yr | Month 6+ |
| **DeraBlog Cloud** (managed hosting) | $9–$499/mo | Year 3+ |
| **Enterprise: Multisite Manager** | $299/yr | Year 2+ |
| **Enterprise: SSO (SAML)** | $499/yr | Year 2+ |
| **Enterprise: White-Label** | $199/yr | Year 2+ |
| **Marketplace Commission** | 20–30% | Year 2+ |

---

## 6. Timeline (26 weeks total — 24 weeks to v1.0 launch + 2 weeks stabilization)

> **Scope realism note:** This timeline is aggressive for a solo developer. Phase 1 in particular packs ~18 epics into 6 weeks, and Phase 4 layers Automation Engine, AI integration, premium themes, and onboarding into another 6. The plan is the **target**, not a contract — if velocity falls short, the priority order is:
>
> 1. **Never compromise:** Hooks system, plugin loader, theme system, default theme, content types, SEO basics, security basics, accessibility, RTL, MIT licensing.
> 2. **Slip if needed:** Premium themes (can ship post-v1.0), Plugin marketplace UI (can launch read-only later), AI Multi-Provider's exotic models (start with 2 providers and grow).
> 3. **Defer if needed:** Advanced page builder blocks, full automation template library (ship 3 templates instead of 7), advanced analytics (real-time visitor count can be polled).
>
> The 24-week target is for **launchable v1.0** (everything users need to switch to DeraBlog), not for "every feature in this document". A delivered, bug-light v1.0 in 30 weeks beats a buggy v1.0 in 24.


### Phase 1 (Weeks 1–6): Core Foundation
**Feature deliverables:**
- Single Laravel repo + Docker Compose for services
- Laravel 11 + Filament 3 + Livewire 3 + Tailwind installed
- DB schema (all tables, including security-relevant: `licenses_hash`, `ai_usage_log`, `webhook_replay_log`, `automation_executions`)
- Authentication (Sanctum + Laravel built-in)
- Roles & Permissions (Spatie)
- Posts, Categories, Tags CRUD with TipTap
- Media library (Spatie MediaLibrary)
- Default theme (Blade-based, RTL/dark mode)
- Hooks system + Plugin loader (with signature-verification stub)
- i18n setup (English + Arabic)
- Basic SEO (meta, sitemap, redirects)
- Basic page caching
- Content Importer (initial source format) — fetches via `EgressGuard`

**Security deliverables (modules + gates):**
- Day-1 Setup Checklist executed (branch protection, gitleaks, dependabot, PGP key, signed commits, license-server scaffold)
- `EgressGuard` module live (Module A) — closes F-1, F-2, F-13 (download path)
- `AuthEdge` module live (Module E) — closes F-9, F-10 (default-deny stub), F-11, F-12, F-14
- `ContentSanitizer` baseline live (Module D) — closes F-5 (baseline), F-8
- `ArtifactTrust` interfaces + Minisign public key bundled (Module C stub) — closes F-3 (admin consent flow)
- CSP middleware with per-request nonce
- Phase-1 exit security tests (PHPStan rules, SSRF tests, XSS payload corpus, SVG sanitizer test, channels-callback test) all green in CI

**Milestone:** Internal Alpha (used on Derabia.com)

### Phase 2 (Weeks 7–12): Complete the Toolkit
**Feature deliverables:**
- Full SEO Suite (analyzer, schema templates, internal linking, broken links — link checker uses `EgressGuard` profile=`link_check`)
- Performance Suite (minify, critical CSS, image optimization)
- Security Suite (WAF-lite, malware scanner, activity log)
- Forms Builder (drag-drop) — submissions piped through `ContentSanitizer`
- Backup System (local + S3 BYOK via `EgressGuard` profile=`backup`)
- Newsletter System (SMTP-based) — subscribe gated by Turnstile + per-IP rate limit
- Comments System (threading, reactions)
- Privacy-First Analytics Dashboard
- Multilingual Suite (full features)

**Security deliverables (modules + gates):**
- Per-channel `Broadcast::channel()` callbacks for every Reverb channel — closes F-10 fully
- Newsletter subscription rate-limit + Turnstile + signed double-opt-in token — closes F-14
- `ai_prompt_audit` PII redaction filter
- All five rate-limit policies wired to `AuthEdge::enforceRateLimit()`
- CSP report endpoint (`POST /security/csp-report`) accumulating violations
- Phase-2 exit security tests: rate-limit corpus, Reverb channel-spoofing test, broken-link SSRF test, comment XSS fuzzing — all green

**🚀 Milestone:** Soft Launch (Indie Hackers, Reddit, Twitter)

### Phase 3 (Weeks 13–18): Differentiate
**Feature deliverables:**
- Page Builder (drag-drop visual builder)
- Theme Builder (custom header/footer)
- Membership basic (Stripe BYOK) — webhook signature verification via `ArtifactTrust`
- Editorial Calendar (FullCalendar) — iCal feed via signed URL token
- Custom Post Types UI polish
- DSAR endpoints (`/api/v1/dsar/export`, `/api/v1/dsar/delete`) for GDPR compliance

**Security deliverables (modules + gates):**
- `ContentSanitizer::sanitizePageBuilderBlock()` per-block-type profiles — closes F-5 fully
- HTML block restricted to admin role only
- Stripe webhook signature verification — closes F-6 partial (Stripe path)
- DSAR round-trip test (export + delete) green
- Phase-3 exit security tests: ≥30 XSS payloads fuzzed against every Page Builder block, all blocked

**Milestone:** Beta v0.9

### Phase 4 (Weeks 19–24): The Headline Features
**Feature deliverables:**
- **Automation Engine (Drawflow)** — full visual builder; webhook action uses `EgressGuard` profile=`webhook`
- **AI Features (BYOK)** — 4 providers integration; all calls go through `AIGateway`
- Plugin marketplace UI (read-only) — manifest signed, verified by `ArtifactTrust`
- License Server live (`license.derablog.com`) with hashed keys + signed responses
- Performance polish
- Documentation finalization
- 1–2 Premium Themes (signed with Minisign)
- Onboarding flow / setup wizard

**Security deliverables (modules + gates):**
- `AIGateway` shipped (Module B) — closes F-4 (per-input cap, per-trigger debounce, hard cost ceiling)
- `ArtifactTrust` full verification path (Module C) — closes F-3 fully (signed plugin/theme verification mandatory before activation), F-13 (signed theme updates)
- License-server schema with `key_hash` only (no plaintext) — closes F-7
- Lemon Squeezy webhook HMAC verification — closes F-6
- AI prompt-injection corpus (≥30 payloads) tested against every AI capability
- Marketplace manifest tampering test green
- Phase-4 exit security tests: cost-ceiling test, signed-plugin verification test, webhook forgery test, license-key-hashing migration test — all green

**🚀 Milestone:** Full Launch v1.0 (Product Hunt, Hacker News, YouTube)

### Phase 5 (Weeks 25–26): Stabilize
**Feature deliverables:**
- Bug fixes from launch feedback
- Documentation polish
- Premium themes finalization
- v1.0.0 stable tag

**Security deliverables (gates):**
- Re-run audit prompt against implemented system; record delta
- In-house pen-test on top-3 surfaces (License Server, AIGateway, Plugin signature verification)
- All P0/P1 bugs from launch closed
- Public security advisory page live; synthetic disclosure round-trip via PGP works
- Phase-5 exit: zero open P0/P1 security issues; v1.0.0 Docker images signed and published

---

## 7. Repository Structure

### Public Repos
```
github.com/derabia/DeraBlog              ← MIT, Core CMS
github.com/derabia/derablog-docs         ← MIT, Documentation site
```

### Private Repos
```
github.com/derabia/derablog-website            ← Marketing site
github.com/derabia/derablog-license-server     ← License validation API
github.com/derabia/derablog-theme-magazine     ← Premium theme
github.com/derabia/derablog-theme-newsroom     ← Premium theme
github.com/derabia/derablog-theme-portfolio    ← Premium theme
github.com/derabia/derablog-theme-multilingual ← Premium theme
github.com/derabia/derablog-theme-minimal     ← Premium theme
```

### Core Repo Layout
```
DeraBlog/
├── app/
│   ├── DeraBlog/                # Core SDK (Hooks, ContentTypes, Plugins, Themes, AI, Automation)
│   ├── Filament/                # Admin resources
│   ├── Http/Controllers/        # Public site controllers
│   ├── Livewire/                # Livewire components
│   ├── Models/
│   └── Providers/
├── bootstrap/
├── config/
├── database/
│   ├── migrations/
│   ├── seeders/
│   └── factories/
├── docker/
├── docs/
├── plugins/                     # Free plugins live here
├── public/
├── resources/
│   ├── views/
│   │   ├── layouts/
│   │   ├── livewire/
│   │   └── themes/
│   ├── css/
│   └── js/                      # Alpine + minimal JS
├── routes/
├── tests/
├── themes/                      # Theme directory
│   └── default/
├── docker-compose.yml
├── docker-compose.prod.yml
├── .env.example
├── README.md
├── LICENSE
├── CHANGELOG.md
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── SECURITY.md
├── PLAN.md
├── ARCHITECTURE.md
└── BUSINESS.md
```

---

## 8. Success Criteria for v1.0

- [ ] Solo author can write, publish, and manage posts via Filament admin.
- [ ] Public site renders posts via Blade with Core Web Vitals all green.
- [ ] SEO output is complete and standards-compliant.
- [ ] Multi-language working (English + Arabic with full RTL).
- [ ] Custom content types definable via admin (e.g., "Recipe").
- [ ] Page Builder lets non-developers build pages visually.
- [ ] At least one example free plugin installable via admin.
- [ ] Default theme is gorgeous on mobile + desktop, supports dark mode and RTL.
- [ ] Editorial Calendar with drag-drop scheduling functional.
- [ ] Automation Engine with at least 5 pre-made templates.
- [ ] AI Features working with all 4 providers (OpenAI, Deepseek, Gemini, OpenRouter).
- [ ] Docker Compose installs full stack on a fresh VPS in under 5 minutes.
- [ ] One-line install script (`curl ... | bash`) functional.
- [ ] Documentation covers: install, theme dev, plugin dev, AI setup, basic admin usage.
- [ ] Public GitHub repo with MIT LICENSE, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY.

---

## 9. Out of Scope for v1.0

- E-commerce (deferred to v1.1 or community plugin)
- Multisite (Enterprise plugin, Year 2)
- Marketplace UI (read-only browser only in v1.0)
- Cloud hosting (Year 3+)
- Native mobile app (REST API ready earlier; app comes later)
- Plugin auto-updates from external repos (Phase 2 of marketplace)
- Real-time collaborative editing (deferred)
- A/B testing framework (deferred)
- Heat maps / Session recordings (deferred)

---

## 10. Year-1 Success Metrics

| Metric | Target |
|---|---|
| GitHub Stars | 1,000+ |
| Active Installs | 500+ |
| Active Sites (telemetry opt-in) | 200+ |
| Twitter/X Followers | 2,000+ |
| Migrations from other platforms | 50+ |
| Premium Theme Sales | $5K–$30K total |
| Community Plugin/Theme submissions | 10+ |
| Newsletter Subscribers | 1,000+ |

---

## 11. Strategic Principles

1. **Core is free, period.** Never paywall a feature that should be in core.
2. **Complete out of the box.** If a modern publisher needs it, it's in the Core.
3. **BYOK for paid services.** AI, S3 backups, payment processing — user provides their key, feature is free.
4. **Plugin / theme dev experience is the moat.** Easier to build → larger ecosystem → harder to displace.
5. **Self-hosted first, Cloud second.** Don't build SaaS until self-hosted is proven.
6. **Adoption before revenue.** Year 1 = users. Year 2+ = revenue.
7. **Privacy by default.** No external trackers, no GDPR cookie banners.
8. **Documentation is product.** Plugin/theme dev docs are first-class deliverables.
