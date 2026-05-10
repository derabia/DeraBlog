# Changelog

All notable changes to DeraBlog will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial repository structure with planning documents.
- `PLAN.md` — Master development plan covering vision, locked decisions, four pillars, ~200 core features across 19 areas, premium strategy, 26-week timeline (24 weeks to v1.0 + 2 weeks stabilization), and success metrics.
- `ARCHITECTURE.md` — Technical architecture covering Laravel TALL Stack, Hooks System, Content Types, Themes, Plugins, License Server, AI Multi-Provider, Editorial Calendar, Automation Engine, SEO subsystem, i18n, caching, security, deployment.
- `BUSINESS.md` — Business strategy covering positioning, hybrid licensing, revenue streams, premium themes plan, marketing strategy, risks, and success metrics.
- `README.md` — Project overview with feature list and installation instructions.
- `LICENSE` — MIT License for Core CMS.
- `LICENSE-COMMERCIAL.md` — Commercial EULA template for Premium Themes.
- `CONTRIBUTING.md` — Contribution guidelines.
- `CODE_OF_CONDUCT.md` — Community Code of Conduct.
- `SECURITY.md` — Security policy and vulnerability disclosure process.
- `.gitignore`, `.gitattributes`, `.editorconfig` — Repository configuration.

### Notes
- Project is **pre-alpha**. No code has been implemented yet — only planning documents.
- The 26-week development timeline begins after the planning phase concludes.
- Soft Launch is targeted for Week 12 (after Phase 2 of development).
- Full Launch (v1.0) is targeted for Week 24 (after Phase 4 of development).
- Stabilization period covers Weeks 25–26 (Phase 5).

### Architectural decisions resolved during deep planning review
- Posts and Pages are content types in the unified `content_entries` table (no separate `posts` table).
- `content_entries` includes denormalized columns (`title`, `excerpt`, `featured_image_id`, `layout`) alongside the `data` JSON for fast queries and clear ownership of layout vs fields.
- Page Builder and themes have explicit rendering responsibility split: theme owns chrome, Page Builder owns content, legacy/imported entries fall back to rich-text rendering.
- Authentication uses two channels (session for web/admin/Livewire, Sanctum tokens for REST API) on the same `User` model.
- Real-time features run on Laravel Reverb with documented use cases and graceful degradation when disabled.
- Automation Engine flows execute on Laravel Queue (Redis-backed) with retry, dead-letter, and per-node execution logging.
- Plugin marketplace data sourced from a separate hosted API (`marketplace.derablog.com`); not required for DeraBlog to function.
- Search Console integration is BYOK (user provides Google OAuth) — consistent with the no-vendor-lock-in philosophy.
- Scope realism note published in `PLAN.md` Section 6: slip-priority order if velocity falls short.

[Unreleased]: https://github.com/derabia/DeraBlog/commits/main
