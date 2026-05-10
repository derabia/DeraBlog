# DeraBlog — Business Strategy

> Owned by Derabia.com. Strategy: **Value-first, Adoption > Revenue (Year 1).**

---

## 1. Product Positioning

**One-sentence pitch:**
> DeraBlog is a modern, fast, SEO-first content management system — open source, self-hosted, and complete out of the box.

**The Hook:**
> *Everything a modern publisher needs, included in the Core, free forever.*

**Differentiators:**
- **Complete out of the box.** No essential capability requires a paid extension. Content management, SEO, performance, security, multilingual, automation, AI, page building, membership, newsletter, analytics, backups — all in the Core.
- **Lightweight Laravel TALL Stack** — runs on entry-level VPS hosting, even shared hosting with PHP 8.2+.
- **No vendor lock-in.** Every external integration is optional and uses Bring-Your-Own-Key (BYOK) — the user owns their accounts.
- **AI-ready with BYOK.** Four providers (OpenAI, Deepseek, Gemini, OpenRouter) integrated; the user supplies their own key and pays only for their own usage.
- **Privacy by default.** Built-in analytics require no external trackers; no cookie banner needed.
- **First-class right-to-left support.** Arabic-first quality, Hebrew and Persian fully supported.
- **One-line VPS install** via Docker Compose.
- **Built-in content importer** for migrating from existing platforms.

---

## 2. Hybrid Licensing Model

DeraBlog uses a **dual-license model**:

### Core CMS — MIT License

**What it allows (anyone):**
- Free for personal use.
- Free for commercial use (own sites, agency client work).
- Free to fork, modify, redistribute, and resell modified versions.
- Source code is public on GitHub.
- No royalties, no restrictions.

**Why MIT?**
- Maximum adoption — removes all friction.
- Builds community trust (no copyleft strings).
- The competitive moat is **value delivered**, not license restrictions.
- Plugin and theme developers can build commercial products without license-conflict concerns.

### Premium Themes — Commercial EULA

**What customers get:**
- Full source code (PHP, Blade, no encryption) — readable, debuggable.
- License key tied to one production domain.
- One year of updates and support, renewable annually.
- 14-day refund window.

**What customers cannot do:**
- Redistribute the source code.
- Resell or sublicense.
- Remove license-key validation logic.
- Use a single license on multiple production domains.

**Protection mechanism:**
- License key validation via `license.derablog.com`.
- Domain locking enforced on activation.
- Update gating — no updates without an active license.
- Strong EULA + active legal enforcement on confirmed violations.

**Why no encryption?**
- Encryption breaks debugging for customers.
- Source-encryption schemes for PHP have been reverse-engineered repeatedly.
- License-key + EULA + domain-lock is industry standard for source-available commercial software.
- Lower customer support burden (no encoder loaders to install).
- Better hosting compatibility.

---

## 3. Revenue Streams

### Phase 1 (Months 1–12) — Adoption-First

| Stream | Pricing | Year 1 Target |
|---|---|---|
| **Core CMS** | **$0 (free, open source)** | 5,000+ active installs |
| **Premium Themes** | $49–$99 each | 5 themes shipped, $5K–$30K total |
| **Custom Development** | $80–$150/hr | On request |
| **Priority Support** | $99–$499/yr | Limited slots |

**Year 1 revenue projection:**
- M1–M3: $0 (Phase 1, build foundation)
- M4–M6: $200–$1,000/mo (Soft Launch + first premium theme)
- M7–M9: $1,000–$3,000/mo (more themes shipped, growing audience)
- M10–M12: $3,000–$10,000/mo (Full Launch + word-of-mouth)
- **Year 1 total: $20K–$60K** (modest, intentional)

### Phase 2 (Year 2–3) — Marketplace & Enterprise

| Stream | Model |
|---|---|
| Marketplace commission | 20–30% of third-party sales |
| Verified Developer Program | $99/year per developer |
| **Enterprise: Multisite Manager** | $299/year |
| **Enterprise: SSO (SAML/OAuth)** | $499/year |
| **Enterprise: White-Label** | $199/year |
| **Enterprise: Compliance pack** | $999+/year |
| Support: Basic | $99/mo (community + email) |
| Support: Pro | $299/mo (priority + Slack) |
| Support: Enterprise | $999+/mo (SLA + custom) |

**Year 2–3 projection:** $20K–$100K/mo if marketplace and enterprise gain traction.

### Phase 3 (Year 3+) — DeraBlog Cloud (Managed Hosting)

| Tier | Price | Includes |
|---|---|---|
| Free | $0 | Subdomain, ad supported |
| Starter | $9/mo | Custom domain, 1 site |
| Pro | $29/mo | 3 sites, advanced features |
| Business | $99/mo | 10 sites, e-commerce |
| Enterprise | $499+/mo | White-label, SLA |

**Year 3+ projection:** $50K–$500K/mo if cloud gains traction.

---

## 4. Premium Themes Plan

| Theme | Niche | Price | Target Launch |
|---|---|---|---|
| Magazine | News and magazine sites | $79 | Month 4 |
| Newsroom | Daily news with a breaking-news ticker | $89 | Month 6 |
| Portfolio | Designers and agencies | $69 | Month 7 |
| Multilingual Blog | RTL-first, Arabic and Hebrew | $99 | Month 8 |
| Minimal | Personal bloggers | $49 | Month 10 |

---

## 5. Marketing Strategy

### Phase 1: Pre-Launch (Month 1)
- Public GitHub repo from day 1 — silent until soft launch.
- Build in Public on Twitter/X.
- Open issues and roadmap visible.

### Phase 2: Soft Launch (Month 3, after development Phase 2)
- Indie Hackers community post.
- Reddit posts in relevant communities (r/SelfHosted, r/PHP, r/laravel, r/webdev).
- Personal network outreach.
- Limited audience, focused on collecting feedback.

### Phase 3: Full Launch (Month 6, after development Phase 4)
- Product Hunt launch.
- Hacker News "Show HN" post with metrics.
- Articles on developer blogs (Dev.to, Hashnode).
- 10-minute walkthrough video.
- Press outreach to relevant publications.

### Phase 4: Growth (Months 7–12)
- Premium theme launches with case studies.
- Email list to existing free users.
- Affiliate program (20–30% commission).
- Migration guides for users coming from other platforms.
- "Why I switched" testimonials.

### Phase 5: Content Marketing (ongoing)
- Blog content on Derabia.com.
- YouTube tutorials and "Building DeraBlog" series.
- Plugin and theme development tutorials (recruits ecosystem).

### Paid Marketing (later, only when conversion data is clear)
- Sponsorships in developer and publishing newsletters.
- Targeted ads on Twitter/Reddit.

---

## 6. Risks & Mitigations

| Risk | Mitigation |
|---|---|
| **No revenue with Open Core MIT** | Premium themes provide visual/design value justifying cost; Year 1 is adoption-focused; Cloud + Enterprise are real revenue sources from Year 2+ |
| **Competition from established CMS platforms** | Position for greenfield projects; built-in importer reduces migration friction; target underserved niches (RTL languages, agencies, privacy-conscious users) |
| **Solo dev burnout / scope creep** | Strict 24-week MVP; firm Phase boundaries; resist scope creep; document everything |
| **Stack lock-in to Laravel** | Laravel is mature and growing; the TALL Stack is officially supported and well-resourced |
| **AI provider changes pricing/API** | Four-provider strategy (OpenAI, Deepseek, Gemini, OpenRouter) hedges against any single provider; BYOK means we don't pay for usage |
| **License key system gets cracked** | Acceptable risk — most paying customers are honest; legal action against confirmed mass violators; key revocation supported |
| **License confusion (MIT vs Commercial)** | Crystal-clear FAQ; separate `LICENSE` (Core MIT) and `LICENSE-COMMERCIAL.md` (themes) files; in-admin badge showing license status |
| **Payment processor outage** | Fallback processor option (deferred to Phase 2) |
| **Lack of plugin ecosystem at launch** | Core itself is feature-rich; ecosystem grows post-launch organically; Verified Developer Program (Year 2) accelerates |

---

## 7. Success Metrics

### Year 1
- 1,000+ GitHub stars
- 500+ active installs
- 200+ active sites (telemetry opt-in)
- $20K–$60K total revenue (premium themes only)
- 5 themes shipped
- 10+ third-party developers experimenting with the SDK
- 50+ migrations from other platforms
- 1,000+ newsletter subscribers

### Year 2
- 10,000+ GitHub stars
- 5,000+ active installs
- $250K–$500K total revenue (themes + enterprise + first cloud beta)
- Marketplace launched with 50–100 sellers
- First $10K month from a single premium theme

### Year 3
- 50,000+ GitHub stars
- 25,000+ active installs
- $1M+ ARR
- DeraBlog Cloud public launch
- Established as a top self-hosted CMS by mindshare

---

## 8. Strategic Principles

1. **Core is free, period.** Never paywall a feature that should be in core. The free experience must stand on its own.
2. **Complete out of the box.** Essential capabilities for a modern publisher belong in the Core, not in paid add-ons.
3. **Adoption before revenue.** Year 1 = users. Year 2+ = revenue.
4. **BYOK for paid services.** AI, S3 backups, payment processing — the user provides their key, the feature stays free.
5. **Plugin / theme dev experience is the moat.** Easier to build → larger ecosystem → harder to displace.
6. **Self-hosted first, Cloud second.** Don't build SaaS until the self-hosted product is proven.
7. **Privacy by default.** No external trackers, no cookie banners required out of the box.
8. **Documentation is product.** Plugin and theme developer docs are first-class deliverables.
9. **No dark patterns.** No artificial limits, no upsell harassment, no telemetry without consent.
10. **Open communication.** Public roadmap, public issues, public discussions.
