# DeraBlog — Architecture

> Technical architecture and design decisions for DeraBlog v1.0.
> **Stack:** Laravel 11 + Livewire 3 + Alpine.js + Blade + Tailwind CSS + MySQL 8 + Redis (TALL Stack).

---

## 1. High-Level Architecture

```
                     ┌──────────────────┐
                     │     Visitor      │
                     │  (browser/bot)   │
                     └────────┬─────────┘
                              │ HTTPS
                     ┌────────▼─────────┐
                     │     Nginx        │
                     │  (Reverse Proxy) │
                     │  + SSL (Certbot) │
                     └────────┬─────────┘
                              │
                              ▼
                     ┌──────────────────┐
                     │  PHP-FPM         │
                     │  Laravel 11      │
                     │  ─────────────   │
                     │  • Public site   │
                     │    (Blade +      │
                     │     Livewire)    │
                     │  • Admin panel   │
                     │    (Filament 3)  │
                     │  • REST API      │
                     │    (Sanctum)     │
                     │  • License check │
                     │  • Hooks system  │
                     │  • AI proxy      │
                     │  • Automations   │
                     └─┬────────┬───────┘
                       │        │
              ┌────────┘        └────────┐
              ▼                          ▼
        ┌──────────┐               ┌──────────┐
        │  MySQL   │               │  Redis   │
        │  (Data)  │               │  (Cache, │
        │          │               │   Queue, │
        │          │               │   Session│
        └──────────┘               └────┬─────┘
                                        │
                                        ▼
                                 ┌─────────────┐
                                 │   Reverb    │
                                 │ (WebSocket) │
                                 └─────────────┘
                       │
              ┌────────┘
              ▼
        ┌──────────┐         ┌─────────────────────┐
        │ Storage  │         │  External Services  │
        │ (Local/  │         │  (BYOK, optional)   │
        │   S3)    │         │  • OpenAI           │
        └──────────┘         │  • Deepseek         │
                             │  • Gemini           │
                             │  • OpenRouter       │
                             │  • Stripe           │
                             │  • SMTP server      │
                             │  • S3/Backblaze     │
                             └─────────────────────┘
```

**Routing rules (Nginx):**
- `/` → Laravel (PHP-FPM)
- `/admin` → Laravel (Filament)
- `/api/*` → Laravel (REST API)
- `/storage/*` → Local files (or proxy to S3)
- `/livewire/*` → Laravel (Livewire endpoint)

**One process model:** Everything is Laravel. No separate Node.js server.

---

## 2. Backend Architecture (Laravel)

### Layered Structure

```
app/
├── DeraBlog/                    # Core extensibility (DeraBlog SDK)
│   ├── Hooks/
│   │   ├── HookManager.php      # Actions + Filters registry
│   │   └── Facades/DeraHooks.php
│   ├── ContentTypes/
│   │   ├── ContentTypeRegistry.php
│   │   ├── ContentTypeBuilder.php
│   │   └── Field/                # Field type definitions
│   ├── Plugins/
│   │   ├── PluginManager.php
│   │   ├── PluginLoader.php
│   │   └── Manifest.php
│   ├── Themes/
│   │   ├── ThemeManager.php
│   │   ├── ThemeRegistry.php
│   │   └── TemplateResolver.php
│   ├── Seo/
│   │   ├── SeoAnalyzer.php
│   │   ├── SchemaBuilder.php
│   │   └── SitemapBuilder.php
│   ├── PageBuilder/
│   │   ├── BlockRegistry.php
│   │   ├── Renderer.php
│   │   └── Blocks/
│   ├── Calendar/
│   │   └── EditorialCalendar.php
│   ├── Automation/
│   │   ├── FlowEngine.php
│   │   ├── TriggerRegistry.php
│   │   ├── ActionRegistry.php
│   │   ├── Triggers/
│   │   └── Actions/
│   ├── AI/
│   │   ├── AIManager.php
│   │   ├── Providers/
│   │   │   ├── OpenAIProvider.php
│   │   │   ├── DeepseekProvider.php
│   │   │   ├── GeminiProvider.php
│   │   │   └── OpenRouterProvider.php
│   │   ├── Services/             # Content writer, SEO suggester, etc.
│   │   └── CostTracker.php
│   ├── Newsletter/
│   ├── Forms/
│   ├── Membership/
│   ├── Backup/
│   ├── Security/
│   └── Cache/
├── Filament/                     # Admin panel (Livewire-based)
│   ├── Resources/
│   ├── Pages/
│   └── Widgets/
├── Http/
│   ├── Controllers/
│   │   ├── Site/                 # Public-facing
│   │   └── Api/                  # REST API
│   └── Middleware/
├── Livewire/                     # Public-site Livewire components
├── Models/
└── Providers/
    ├── AppServiceProvider.php
    ├── DeraBlogServiceProvider.php
    └── PluginServiceProvider.php
```

### Why this split?

The `DeraBlog/` namespace is the **Core SDK**. It's intentionally separate from `Filament/`, `Http/`, `Models/` so:
- Plugins use it without coupling to Laravel internals.
- Parts of it can be extracted as Composer packages later.
- Code is auditable and clear for plugin/theme developers.

---

## 3. The Hooks System

### Concept

Modeled after WordPress's `do_action`/`apply_filters`. Two primitives:

- **Action:** "Run this code when X happens." (Side-effect)
- **Filter:** "Modify this value before it's used." (Pure transformation)

### Implementation Sketch

```php
namespace App\DeraBlog\Hooks;

class HookManager
{
    protected array $actions = [];
    protected array $filters = [];

    public function addAction(string $hook, callable $cb, int $priority = 10): void;
    public function doAction(string $hook, mixed ...$args): void;
    public function addFilter(string $hook, callable $cb, int $priority = 10): void;
    public function applyFilters(string $hook, mixed $value, mixed ...$args): mixed;
}
```

### Standard Hook Names (initial set)

**Actions:**
- `post.creating`, `post.created`, `post.updating`, `post.updated`, `post.publishing`, `post.published`, `post.deleting`, `post.deleted`
- `user.registered`, `user.logged_in`, `user.logged_out`
- `media.uploaded`, `media.deleted`
- `comment.created`, `comment.approved`
- `plugin.activated`, `plugin.deactivated`
- `theme.activated`
- `automation.triggered`, `automation.completed`
- `ai.request_made`, `ai.response_received`

**Filters:**
- `post.title`, `post.content`, `post.excerpt`, `post.meta`
- `seo.meta_title`, `seo.meta_description`, `seo.og_image`
- `sitemap.entries`, `sitemap.priority`
- `feed.items`, `feed.title`
- `template.path`, `template.context`
- `view.data`
- `ai.prompt`, `ai.response`

---

## 4. Content Types & Field Builder

### Schema

```sql
content_types
├── id, name, slug, label, label_plural
├── icon, supports_seo, supports_translations
├── public, hierarchical, has_archive
└── timestamps

content_type_fields
├── id, content_type_id, name, label, type
├── required, multiple, default_value
├── settings (JSON)
├── conditional_logic (JSON)
├── order
└── timestamps

content_entries
├── id, content_type_id, author_id, parent_id
├── slug, status, published_at
├── data (JSON)
└── timestamps

content_entry_translations
├── id, content_entry_id, locale
├── data (JSON)
└── timestamps
```

### Default Content Type: `post`

Built-in. Provides standard blog post functionality. Other types added via admin UI.

### Field Types (15)

| Type | Description | Storage |
|---|---|---|
| `text` | Single line | string |
| `textarea` | Multi-line | text |
| `richtext` | TipTap editor | JSON |
| `number` | Numeric | number |
| `boolean` | Yes/No | bool |
| `select` | Dropdown | string |
| `multiselect` | Multi-dropdown | array |
| `image` | Single image | media_id |
| `gallery` | Multi image | array of media_id |
| `file` | File upload | media_id |
| `date` | Date picker | datetime |
| `color` | Color picker | string |
| `url` | URL input | string |
| `relation` | Reference to another entry | id or array |
| `repeater` | Array of subfields | array of objects |
| `flexible_content` | Mixed block types | array |
| `clone` | Reuse field group | array |

---

## 5. Theme System

### Theme Structure

```
themes/default/
├── theme.json                    # Manifest
├── assets/
│   ├── tailwind.config.js
│   ├── styles/
│   ├── scripts/                  # Alpine components
│   └── images/
├── views/                        # Blade templates
│   ├── layouts/
│   │   └── app.blade.php
│   ├── home.blade.php
│   ├── post.blade.php
│   ├── category.blade.php
│   ├── author.blade.php
│   ├── search.blade.php
│   ├── 404.blade.php
│   └── partials/
├── slots/
│   └── slots.php
└── settings.php
```

### theme.json

```json
{
  "name": "default",
  "version": "1.0.0",
  "label": "DeraBlog Default",
  "description": "The default DeraBlog theme.",
  "author": "Derabia.com",
  "license": "MIT",
  "screenshot": "screenshot.png",
  "supports": ["dark-mode", "rtl", "blocks", "page-builder", "custom-fonts"],
  "slots": ["header", "footer", "sidebar", "post-after-content", "post-sidebar"],
  "settings": {
    "primary_color": { "type": "color", "default": "#3B82F6" },
    "font_family": { "type": "select", "options": ["Inter", "Cairo", "Roboto"], "default": "Inter" },
    "show_author_box": { "type": "boolean", "default": true }
  }
}
```

### Template Hierarchy

When rendering a Post page, Laravel looks for templates in this order:
1. `themes/{active}/views/post-{slug}.blade.php`
2. `themes/{active}/views/post-{contentType}.blade.php`
3. `themes/{active}/views/post.blade.php`
4. `themes/default/views/post.blade.php` (fallback)

---

## 6. Page Builder

A drag-drop visual builder modeled after Elementor, built with Livewire + Alpine.

### Architecture

```
PageBuilder
├── BlockRegistry           # Registers available blocks
├── Renderer                # Renders saved layout to HTML
└── Blocks/
    ├── Heading.php
    ├── Text.php
    ├── Image.php
    ├── Button.php
    ├── Section.php
    ├── Column.php
    ├── Spacer.php
    ├── Form.php
    ├── PostList.php
    └── ... (more)
```

### Block Definition

Each block is a PHP class implementing `BlockContract`:

```php
class HeadingBlock implements BlockContract
{
    public function name(): string { return 'heading'; }
    public function label(): string { return 'Heading'; }
    public function icon(): string { return 'heroicon-o-document-text'; }
    public function fields(): array { return [...]; }
    public function render(array $data): string;
}
```

### Storage

Page Builder layouts stored as JSON in the post's `layout` column:

```json
{
  "blocks": [
    {
      "type": "section",
      "props": { "padding": "py-16", "background": "#f8fafc" },
      "children": [
        {
          "type": "heading",
          "props": { "text": "Welcome", "level": "h1", "align": "center" }
        }
      ]
    }
  ]
}
```

### Theme Builder

Special variant of Page Builder for header/footer/archive layouts. Stored in `theme_layouts` table.

---

## 7. Plugin System

### Plugin Structure

```
plugins/
└── example-plugin/
    ├── plugin.json
    ├── src/
    │   ├── ServiceProvider.php
    │   ├── Livewire/
    │   ├── Filament/
    │   ├── Hooks/
    │   └── Services/
    ├── routes/
    ├── database/migrations/
    ├── resources/
    │   ├── views/
    │   └── lang/
    ├── public/
    ├── composer.json
    └── README.md
```

### plugin.json

```json
{
  "name": "example-plugin",
  "version": "1.0.0",
  "label": "Example Plugin",
  "description": "Description here.",
  "author": "Author Name",
  "license": "MIT",
  "license_key_required": false,
  "requires": {
    "deraBlog": ">=1.0.0",
    "php": ">=8.2"
  },
  "service_provider": "ExamplePlugin\\ServiceProvider",
  "permissions": ["read_posts"],
  "hooks": {
    "actions": ["post.published"],
    "filters": ["post.content"]
  }
}
```

### Loading

1. `PluginLoader` scans `plugins/` at boot.
2. For each enabled plugin, reads `plugin.json`.
3. If `license_key_required: true`, validates via `LicenseClient` (24h cache).
4. Registers `service_provider` with Laravel container.
5. Plugin's `boot()` runs.
6. Migrations run automatically on first activation.

---

## 8. AI Multi-Provider Architecture

### Concept

A unified interface to multiple AI providers, with the user supplying their own API key (BYOK). The Core supports 4 providers from day 1; each Service (Content Writer, Translator, etc.) can use any of them.

### Architecture

```
┌─────────────────────────────────────────┐
│  AI Service Layer                       │
│  ─────────────────                      │
│  • ContentWriter                        │
│  • SeoSuggester                         │
│  • Translator                           │
│  • Summarizer                           │
│  • AltTextGenerator                     │
│  • TagSuggester                         │
│  • SchemaGenerator                      │
│  • OutlineGenerator                     │
│  • ContentRepurposer                    │
│  • CommentModerator                     │
│  • ImageGenerator (DALL-E, Imagen, SD) │
└─────────┬───────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────┐
│  AIManager (provider-agnostic)          │
│  ─────────────────                      │
│  • complete($prompt, $opts)             │
│  • embed($text)                         │
│  • image($prompt)                       │
└─────────┬───────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────┐
│  Provider Implementations               │
│  ─────────────────                      │
│  • OpenAIProvider                       │
│  • DeepseekProvider                     │
│  • GeminiProvider                       │
│  • OpenRouterProvider                   │
└─────────┬───────────────────────────────┘
          │
          ▼
    External APIs
```

### Provider Interface

```php
interface AIProviderContract
{
    public function complete(string $prompt, array $options = []): AIResponse;
    public function stream(string $prompt, array $options = []): iterable;
    public function embed(string $text): array;
    public function image(string $prompt, array $options = []): string;
    public function listModels(): array;
    public function calculateCost(int $inputTokens, int $outputTokens, string $model): float;
}
```

### Supported Providers (v1.0)

| Provider | Models | Notes |
|---|---|---|
| **OpenAI** | GPT-4o, GPT-4 Turbo, DALL-E 3, text-embedding-3 | Most popular |
| **Deepseek** | deepseek-chat, deepseek-coder | Cheapest by far ($0.14/1M tokens) |
| **Gemini** | gemini-1.5-pro, gemini-1.5-flash, imagen-3 | Generous free tier |
| **OpenRouter** | 100+ models via single API | Multi-model router |

### Settings UI

In admin → Settings → AI:
```
Provider:        [Dropdown: OpenAI / Deepseek / Gemini / OpenRouter]
API Key:         [Encrypted password input]
Default Model:   [Dropdown — fetched from provider]
Temperature:     [0.0 – 2.0 slider, default 0.7]
Max Tokens:      [Number input, default 2000]
Monthly Limit:   [Optional cost cap, e.g., $50]
Cost This Month: $X.XX (with breakdown)
```

### Cost Tracking

Every AI call records:
- Provider, model
- Input tokens, output tokens
- Estimated cost (per provider's pricing)
- Service used (content_writer, translator, etc.)
- User who triggered it

Stored in `ai_usage_log` table; visible in admin dashboard.

---

## 9. Editorial Calendar

### Concept

Drag-drop editorial calendar for content scheduling, built with Livewire + FullCalendar.js (Alpine wrapper).

### Schema

Reuses `content_entries` (no new tables). Filters by `status` and `published_at`.

### UI

```
┌──────────────────────────────────────────┐
│  Editorial Calendar                       │
│  [Month] [Week] [Day]  [Filters: ▼]      │
│                                           │
│  Mon  Tue  Wed  Thu  Fri  Sat  Sun       │
│  ┌──┐ ┌──┐ ┌──┐                          │
│  │  │ │📝│ │🔵│        ← Drag posts      │
│  └──┘ └──┘ └──┘          between days    │
└──────────────────────────────────────────┘
```

### Features

- Drag-drop to reschedule
- Multi-author filter
- Status filter (Draft / Scheduled / Published / Private)
- Color coding by category/type
- Quick edit from calendar
- Bulk reschedule
- iCal export endpoint (subscribe from Google Calendar)
- Content gap detection (highlight days with no scheduled posts)

---

## 10. Automation Flow Engine

### Concept

Visual node-based automation builder using Drawflow (vanilla JS). Triggers + Conditions + Actions + Logic.

### Architecture

```
┌─────────────────────────────────────┐
│  FlowEngine                          │
│  ─────────────────                   │
│  • register Triggers/Actions         │
│  • execute Flow on Trigger fire      │
│  • handle Conditions, Delays, Loops  │
│  • log execution                     │
└─────────┬───────────────────────────┘
          │
   ┌──────┴──────┐
   │             │
   ▼             ▼
TriggerRegistry  ActionRegistry
   │             │
   │             │
   ▼             ▼
Triggers/        Actions/
├── PostPublished      ├── SendEmail
├── CommentReceived    ├── SendNewsletter
├── NewSubscriber      ├── PostToSocial
├── FormSubmitted      ├── AddTag
├── UserRegistered     ├── UpdateStatus
├── Schedule           ├── CallWebhook
├── Webhook            ├── RunAIAction
└── Manual             ├── SendSlack
                       ├── SendDiscord
                       └── SendTelegram
```

### Flow Storage

Each Flow is stored as JSON in `automations` table:

```json
{
  "name": "Notify Slack on new post",
  "active": true,
  "trigger": { "type": "post.published", "config": {} },
  "nodes": [
    { "id": "1", "type": "trigger", "data": {...} },
    { "id": "2", "type": "condition", "data": { "if": "category == tech" } },
    { "id": "3", "type": "ai_action", "data": { "service": "summarizer" } },
    { "id": "4", "type": "action", "data": { "type": "slack", "webhook": "..." } }
  ],
  "connections": [
    { "from": "1", "to": "2" },
    { "from": "2", "to": "3", "condition": "true" },
    { "from": "3", "to": "4" }
  ]
}
```

### Pre-made Templates

Shipped with the Core:
- "Notify Slack on new post"
- "Tweet new posts automatically"
- "Weekly newsletter from latest posts"
- "AI-generate excerpt before publish"
- "Cross-post to Mastodon + LinkedIn"
- "Email me on new comment"
- "Auto-spam filter via AI"

---

## 11. License Server (Premium Themes)

A separate Laravel application running on `license.derablog.com` for Premium Theme validation.

### Schema (License Server side)

```sql
licenses
├── id, key (DRBL-XXXX-XXXX-XXXX format)
├── product_id (FK → products)
├── customer_email
├── lemon_squeezy_order_id
├── activated_at, expires_at, deactivated_at
├── max_domains (default: 1)
└── timestamps

license_activations
├── id, license_id
├── domain, ip_address, user_agent
├── activated_at, last_check_at
└── timestamps

products
├── id, slug, name, type ('theme' for v1)
├── version_latest, download_url
└── timestamps
```

### Validation Flow

1. DeraBlog → POST `/api/v1/validate` `{ license_key, domain, product_slug, version }`
2. License server checks: key valid? not expired? domain matches?
3. Returns: `{ valid, expires_at, latest_version, update_url }`
4. DeraBlog caches result for 24 hours.

### No Encryption

Premium themes ship as plain code (Blade + PHP + assets). Protection via:
- License key validation
- Domain locking
- Update gating
- Strong EULA

---

## 12. SEO Subsystem

### Schema

```sql
seo_meta  (polymorphic)
├── id, seoable_type, seoable_id
├── locale
├── meta_title, meta_description
├── og_title, og_description, og_image_id
├── twitter_card, twitter_image_id
├── canonical_url
├── robots (JSON)
├── focus_keywords (JSON array)
├── readability_score, seo_score
├── schema_overrides (JSON)
└── timestamps

redirects
├── id, from_path, to_path, code (301/302), is_regex
└── timestamps
```

### SEO Pipeline (per page)

1. Resolve content (post, category, etc.).
2. Lookup `seo_meta` for content + locale.
3. Apply filters: `seo.meta_title`, `seo.meta_description`, etc.
4. Build JSON-LD (Article, BreadcrumbList, Person, Organization, WebSite, FAQ, Recipe, Product, etc.).
5. Build OG + Twitter tags.
6. Build canonical + hreflang.
7. Render via Blade `@stack('seo')` directive.

### Yoast-like Analyzer (Core, Full Features)

Checks:
- Multiple focus keywords in title, slug, first paragraph, headings, meta description.
- Title length (50–60 chars), meta description (140–160 chars).
- Image alt texts.
- Internal/external links.
- Readability (Flesch reading ease).
- Subheading distribution.
- Keyword density.
- Cornerstone content cross-linking.

Returns score + actionable suggestions, displayed as a Livewire widget in the post editor.

---

## 13. i18n Strategy

- **Default locale:** English (`en`)
- **Supported locales:** Configurable. RTL for Arabic, Hebrew, Persian.
- **URL strategy:** Prefix-based (`/en/`, `/ar/`). Default locale optionally prefixless.
- **DB content:** `spatie/laravel-translatable` (JSON storage in models).
- **UI strings:** Laravel `lang/` files.
- **Theme translations:** `themes/{active}/lang/{locale}/messages.php`.
- **Plugin translations:** `plugins/{slug}/resources/lang/{locale}/messages.php`.
- **Direction switch:** Auto-injected `dir="rtl"` on `<html>` for RTL locales.
- **Tailwind RTL:** `tailwindcss-rtl` plugin for logical properties.
- **Translation Memory:** Local PHP-based memory (no external API), reuses translated strings across content.

---

## 14. Caching Strategy

| Layer | Tool | TTL |
|---|---|---|
| Full-page HTML cache | Laravel Cache + middleware | 300s for posts, 60s for index |
| API responses | Redis | 60s, invalidated by hooks |
| DB queries | Laravel cache (Redis) | 600s |
| Livewire computed | Built-in | per-request |
| Static assets | Nginx | 30 days, hashed filenames |
| Images | Image optimizer + CDN | Permanent (hashed) |
| License validation | Local file cache | 24h |
| AI responses (deduplication) | Redis | 1h (configurable) |

**Invalidation:**
- `post.published` action triggers cache purge: post URL, category URL, home URL, sitemap.
- Bulk operations invalidate via tag (`posts`, `categories`, etc.).

---

## 15. Security

- **CSRF:** Laravel default + Sanctum for API.
- **XSS:** Blade auto-escape + HTMLPurifier wrapper for TipTap output.
- **SQL Injection:** Eloquent ORM, no raw queries.
- **Rate Limiting:** Laravel RateLimiter on auth, comments, search, AI calls, license validation.
- **CAPTCHA:** Cloudflare Turnstile (free) or Honeypot built-in.
- **HTTPS:** Enforced via Nginx (Let's Encrypt).
- **Headers:** CSP, HSTS, X-Frame-Options, X-Content-Type-Options.
- **Plugin Security:** Plugins request permissions in manifest; admin reviews on install.
- **Livewire Security:** All actions go through CSRF + signed payloads.
- **AI Key Security:** API keys encrypted at rest with Laravel encryption (`Crypt::encryptString`).
- **WAF-lite:** Built-in firewall rules for common attack patterns (SQLi, XSS, path traversal).
- **Brute Force Protection:** Login attempt limiting + IP blocking.
- **2FA:** TOTP-based (Google Authenticator compatible), no SMS.

---

## 16. Deployment

### One-line install on fresh VPS

```bash
curl -fsSL https://get.derablog.com/install | bash
```

The script:
1. Verifies/installs Docker + Docker Compose.
2. Clones the repo.
3. Generates `.env` with secure secrets.
4. Runs `docker compose up -d`.
5. Runs migrations + seeds.
6. Prints admin credentials and URL.

### Production Docker Compose

`docker-compose.prod.yml` includes:
- nginx (with auto-SSL)
- php-fpm (Laravel)
- mysql 8 (persistent volume)
- redis (persistent volume)
- reverb (WebSocket, profile-gated)
- queue worker
- scheduler (Laravel scheduler in cron)

**No Node.js process required for production.** All assets pre-built during image build.

---

## 17. Monitoring & Observability

- **Errors:** Sentry (free tier).
- **Logs:** Laravel logs to file. Optional: ship to Logtail/Better Stack.
- **Uptime:** UptimeRobot (free).
- **Metrics:** Laravel Pulse (free, official) — built-in dashboard for slow queries, queue depth, cache hit ratio.
- **Backups:** `spatie/laravel-backup` runs daily, ships to S3-compatible storage.
- **AI Usage:** Custom dashboard showing token usage, cost, by service/user.
- **Automation Logs:** Every Automation Flow execution logged.

---

## 18. Decisions Deferred to Post-v1.0

- E-commerce (community plugin or v1.1)
- Multisite (Enterprise plugin, Year 2)
- GraphQL API alongside REST
- Native mobile app (REST API ready earlier)
- A/B testing framework
- Real-time collaborative editing
- Heat maps / Session recordings
- Plugin auto-updates from external repos (marketplace Phase 2)
- DeraBlog Cloud (managed SaaS) — Year 3+
