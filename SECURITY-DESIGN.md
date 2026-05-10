# DeraBlog — Security Design

> **Scope:** Authoritative technical security architecture for DeraBlog v1.0.
> **Last updated:** 2026-05-10
> **Companion documents:** `ARCHITECTURE.md` (system architecture), `SECURITY.md` (disclosure policy), `PLAN.md` (roadmap with security gates).

This document is the single source of truth for:
1. The 5 cross-cutting security modules.
2. The resolved security decisions for v1.0.
3. The threat-model assumptions baked into the architecture.
4. The trade-offs we accepted and the residual risks we acknowledge.

---

## 1. Resolved Security Decisions

| Decision | Choice | Rationale | Architectural implication |
|---|---|---|---|
| Plugin / theme code-signing | Minisign (Ed25519), root key on YubiKey, public key bundled with Core | One tool, hardware-backed custody, fits solo-dev ops | `ArtifactTrust::verifyPluginPackage()` gates every activation |
| Egress allow-list | Default-deny private IPs (RFC1918, 169.254/16, 127/8, IPv6 link-local), DNS-rebinding-safe, HTTPS-only for vendor hosts | Closes the SSRF cluster at one chokepoint | Single Guzzle handler stack used everywhere |
| Content Security Policy | Strict-CSP with per-request nonce, `'strict-dynamic'`, no `unsafe-inline` | Only XSS posture that survives Page Builder + plugins + themes | Per-request nonce; cache-key includes nonce placeholder |
| 2FA recovery | Backup codes shown once, hashed (`Hash::make`), single-use; password reset does not bypass 2FA on next login; lost-codes routed to manual support | Removes "email compromise → 2FA bypass" without WebAuthn complexity | `AuthEdge` enforces post-reset 2FA prompt |
| Compliance scope (v1.0) | GDPR-ready by design + Egypt Law 151/2020 (operator residency); PDPL-SA acknowledged; no PCI scope | GDPR is strictest declared scope; satisfying it satisfies the rest | DSAR endpoints, retention windows, processor disclosure UI |
| Security disclosure | `security@derablog.com` + 4096-bit RSA PGP + GitHub private vulnerability reporting; SLA: ack ≤72h, fix Critical ≤7d / High ≤30d / Med ≤90d | Industry standard, zero ongoing cost | Day-1 setup; key fingerprint in `SECURITY.md` |
| Telemetry default | OFF; opt-in during onboarding; aggregate metrics only (PHP version, locale, plugin count); no IPs, no domains | Privacy-by-default is the brand promise | Separate vendor service `telemetry.derablog.com` |
| Marketplace publisher tiers | `Verified` (signed + manually reviewed) and `Community` (unsigned, requires per-install admin consent, no auto-update) | Brand-trust + community-velocity at the same time | `plugin.json` carries `trust_tier` and `signature` |
| AI prompt-log retention | Token counts: 90 days. Prompt content: opt-in per service, 7 days, PII-redacted | Cost tracking needs counts; prompt bodies are PII risk | Two tables: `ai_usage_log`, `ai_prompt_audit` |
| Rate-limit policy | Hardcoded defaults in `config/dera-rate-limits.php`; auth 5/min/IP; comments 10/h/IP; AI 30/min/user with 4K-token input cap; license validation 100/h/key | Concrete numbers ship; tunable per install | One config file, one trait, applied via `RouteServiceProvider` |
| AI cost ceiling | Hard ceiling enforced in `AIGateway` ($50/mo default, env-overridable up to $500, NOT admin-overridable); soft warning at 50% | Reputational ATO from "DeraBlog drained my key" is platform-fatal | `AIGateway::remainingBudget()` checked before every call |
| Plugin trust model | Two-tier: Verified = trust anchor; Community = honor-system + explicit consent + no auto-update | Forced binary choice would either kill community or kill brand | UI badges, update gating, consent dialogs |

---

## 2. Cross-Cutting Security Modules

All security-critical logic lives in **5 modules** under `app/Security/`. PHPStan rules forbid bypassing them.

### Module A — `EgressGuard`

**Closes:** F-1 (Automation Webhook SSRF), F-2 (Importer + Broken-Link Checker SSRF), F-13 partial (theme update channel uses this for downloads).

**Library:** `guzzlehttp/guzzle ^7.9` + custom Guzzle middleware. No third-party SSRF lib.

**Public interface (`app/Security/EgressGuard/OutboundHttpClient.php`):**
```php
namespace App\Security\EgressGuard;

interface OutboundHttpClient
{
    public function get(string $url, string $profile, array $opts = []): Response;
    public function post(string $url, string $profile, array $opts = []): Response;
    public function fetchWithBudget(string $url, string $profile, int $maxBytes, int $timeoutSec, array $opts = []): Response;
}
```

**Where it plugs in:**
- Bound in `App\Providers\SecurityServiceProvider` as `OutboundHttpClient::class`.
- Every PHP outbound HTTP call MUST resolve through this binding.
- PHPStan custom rule (`Larastan` extension) bans direct `Http::` / `Guzzle\Client` / `cURL` use outside `App\Security\EgressGuard\*`.
- Implementation: DNS resolve → IP allow/deny check → reconnect with resolved IP via `CURLOPT_RESOLVE` (TOCTOU-safe) → max-bytes cap → max-time cap → response `Content-Type` allow-list per call site.

**Profile registry (`config/dera-egress.php`):**
```php
return [
    'profiles' => [
        'marketplace' => ['hosts' => ['marketplace.derablog.com'], 'schemes' => ['https'], 'max_bytes' => 10_000_000, 'timeout' => 30],
        'license'     => ['hosts' => ['license.derablog.com'],     'schemes' => ['https'], 'max_bytes' => 64_000,    'timeout' => 10],
        'ai'          => ['hosts' => 'config:dera.ai.allowed_hosts','schemes' => ['https'], 'max_bytes' => 5_000_000, 'timeout' => 60],
        'webhook'     => ['hosts' => '*public-only*',              'schemes' => ['https', 'http'], 'max_bytes' => 1_000_000, 'timeout' => 15],
        'import'      => ['hosts' => '*public-only*',              'schemes' => ['https', 'http'], 'max_bytes' => 50_000_000, 'timeout' => 120, 'mime' => ['image/*', 'video/*', 'application/pdf']],
        'link_check'  => ['hosts' => '*public-only*',              'schemes' => ['https', 'http'], 'method' => 'HEAD', 'timeout' => 10],
        'backup'      => ['hosts' => 'config:dera.backup.s3_endpoints', 'schemes' => ['https'], 'max_bytes' => 5_368_709_120, 'timeout' => 3600],
    ],
    'denied_cidrs' => ['10.0.0.0/8', '172.16.0.0/12', '192.168.0.0/16', '127.0.0.0/8', '169.254.0.0/16', '::1/128', 'fc00::/7', 'fe80::/10'],
];
```

**Phase + week:** Phase 1, Week 2.

---

### Module B — `AIGateway`

**Closes:** F-4 (AI cost amplification + prompt-injection cost).

**Libraries:** `openai-php/laravel ^0.10`, `google-gemini-php/laravel ^2.x`, plus thin HTTP wrappers for Deepseek and OpenRouter (which use OpenAI-compatible APIs).

**Public interface (`app/Security/AIGateway/AIGateway.php`):**
```php
namespace App\Security\AIGateway;

interface AIGateway
{
    public function complete(string $service, string $userInput, array $opts = []): AIResponse;
    public function estimateCost(string $service, int $inputTokens): float;
    public function remainingBudget(): float;
}
```

**Where it plugs in:**
- Single binding consumed by every AI feature service (`ContentWriter`, `Translator`, `Summarizer`, etc.).
- AI feature classes never call provider SDKs directly. PHPStan rule enforces this.
- `complete()` enforces, in order:
  1. Per-input token cap (4K default, configurable per service).
  2. Per-trigger debounce (60s same source row).
  3. Per-IP / per-user rate limit (delegated to `AuthEdge`).
  4. Cost-budget check: hard ceiling, soft warning at 50%.
  5. Provider call.
  6. Token + cost log to `ai_usage_log`.
  7. Output sanitized via `ContentSanitizer` before returning.
  8. Action `ai.request_made` fired via Hooks.

**Hard ceiling logic:**
- Default: $50/month/install.
- `AI_HARD_CEILING_USD` env var can raise up to $500.
- The admin UI cannot raise the ceiling (read-only display).
- Reaching the ceiling disables AI features for the rest of the month with a clear admin notification.

**Phase + week:** Phase 4, Week 19.

---

### Module C — `ArtifactTrust`

**Closes:** F-3 (plugin RCE via marketplace), F-6 (Lemon Squeezy webhook forgery), F-7 (plaintext license keys), F-13 (theme update injection).

**Libraries:**
- `paragonie/sodium_compat ^1.21` (libsodium for Ed25519 verification).
- Native PHP `hash_hmac`, `hash_equals` for HMAC.
- Native PHP `random_bytes` for nonces.

**Public interface (`app/Security/ArtifactTrust/ArtifactVerifier.php`):**
```php
namespace App\Security\ArtifactTrust;

interface ArtifactVerifier
{
    public function verifyPluginPackage(string $zipPath, string $signaturePath): TrustResult;
    public function verifyThemePackage(string $zipPath, string $signaturePath): TrustResult;
    public function verifyWebhookPayload(string $body, string $signatureHeader, string $secret): bool;
    public function verifyMarketplaceManifest(string $manifestJson, string $signature): bool;
    public function hashLicenseKey(string $plaintextKey): string;
}
```

**`TrustResult` carries:**
- `valid: bool`
- `tier: 'verified' | 'community' | 'untrusted'`
- `publisher: string | null`
- `key_id: string | null`
- `error: string | null`

**Where it plugs in:**
- `PluginLoader::activate()` — refuses unsigned plugins unless admin has confirmed the per-install Community consent and `tier === 'community'`.
- `ThemeManager::activate()` — same.
- `LemonSqueezyWebhookController::handle()` — verifies HMAC-SHA256 signature, enforces 5-min replay window via `webhook_replay_log` idempotency-key store.
- `LicenseClient::validate()` — verifies server-signed response (`X-Dera-Signature` header).
- `LicenseServer::issueLicense()` — stores `hash(key)` only; plaintext shown once at checkout.
- `MarketplaceClient::fetchListings()` — verifies signed manifest before display.

**`plugin.json` signature field:**
```json
{
  "name": "seo-pro",
  "version": "1.2.0",
  "service_provider": "...",
  "publisher_identity": "derabia",
  "trust_tier": "verified",
  "signature": {
    "algorithm": "ed25519",
    "public_key_id": "derabia-2026-04",
    "value": "RWRzZXh0Lzc2ZTRkOC4uLg=="
  }
}
```

**Phase + week:**
- Webhook verification + license-key hashing: License Server build (Q3 2026).
- Plugin/theme signature verification: Phase 4, Week 22.
- Day-1: Stub interfaces and Minisign public key committed.

---

### Module D — `ContentSanitizer`

**Closes:** F-5 (Page Builder XSS), F-8 (SVG/polyglot upload).

**Libraries:**
- `mews/purifier ^3.4` (Laravel wrapper around HTMLPurifier 4.18).
- `enshrined/svg-sanitize ^0.21`.
- `intervention/image ^3.7` for re-encoding raster images.

**Public interface (`app/Security/ContentSanitizer/ContentSanitizer.php`):**
```php
namespace App\Security\ContentSanitizer;

interface ContentSanitizer
{
    public function sanitizeRichText(string $html, string $context = 'public'): string;
    public function sanitizeSvg(string $svgXml): ?string;
    public function reencodeImage(UploadedFile $file): ?ProcessedImage;
    public function sanitizePageBuilderBlock(string $blockType, array $blockData): array;
}
```

**Sanitizer profiles (`config/purifier.php`):**

| Profile | Allowed tags | Allowed attrs | Use case |
|---|---|---|---|
| `comment` | p, em, strong, code, blockquote, a (no `target=_blank`) | href (http/https only) | User comments |
| `post` | full HTML5 set (no `<script>`, no `<iframe>`) | class, id, data-*, lang, dir | TipTap-rendered post bodies |
| `pagebuilder` | per-block-type allow-list | per-block-type | Page Builder block render() output |
| `pagebuilder-html-block` | strict allow-list | href, src (https only) | Admin-only HTML block |

**Where it plugs in:**
- `Block::render()` invokes `sanitizePageBuilderBlock()` before emitting HTML.
- `MediaUploadController` calls `sanitizeSvg()` (or rejects per config flag) and `reencodeImage()` for raster types.
- `CommentController::store()` runs `sanitizeRichText($body, 'comment')`.
- TipTap server-render path runs `sanitizeRichText($renderedHtml, 'post')`.
- AI features sanitize their output via `sanitizeRichText` before display.

**Phase + week:** Phase 1, Week 3 (baseline). Page Builder profile: Phase 3, Week 13.

---

### Module E — `AuthEdge`

**Closes:** F-9 (Sanctum lifecycle), F-10 (Reverb channel auth), F-11 (cache-key poisoning), F-12 (2FA recovery flow), F-14 (newsletter subscription bombing).

**Libraries:**
- `laravel/sanctum ^4.0`.
- `pragmarx/google2fa ^8.0`.
- `bacon/bacon-qr-code ^3.0`.
- `laravel/reverb ^1.x`.
- Native `Illuminate\Cache\RateLimiter`.

**Public interface (`app/Security/AuthEdge/AuthEdge.php`):**
```php
namespace App\Security\AuthEdge;

interface AuthEdge
{
    public function issueScopedToken(User $u, array $scopes, int $ttlDays = 30): string;
    public function rotateExpiredTokens(): int;
    public function authorizeChannel(string $channel, ?User $user, array $params = []): bool;
    public function composeCacheKey(Request $r): string;
    public function enforceRateLimit(string $bucket, Request $r): void;
    public function postPasswordResetRequires2FA(User $u): bool;
}
```

**Cache-key composition:**
```php
return implode('|', [
    'v3',                                      // bump on cache-shape change
    $request->getHost(),                       // canonical host post-trusted-proxy
    $request->getPathInfo(),                   // path only, no query unless route opts in
    app()->getLocale(),
    config('dera.theme.active_version'),
    auth()->check() ? 'auth' : 'guest',
    $request->cookies->get('dera_member_tier', 'public'),
    $request->getPreferredLanguage(['en','ar','he','fa','de','fr','es']) ?? 'en',
]);
```
Cache is **skipped entirely** when: response carries `Set-Cookie`; status ≠ 200; request has `Authorization`; route is marked `dera.uncachable`.

**Reverb channels (`routes/channels.php`) — default-deny:**
```php
use App\Security\AuthEdge\AuthEdge;

foreach ([
    'private-user.{userId}',
    'presence-analytics',
    'presence-entry.{entryId}',
    'private-automation.{userId}',
] as $pattern) {
    Broadcast::channel($pattern, fn ($user, ...$args) =>
        app(AuthEdge::class)->authorizeChannel($pattern, $user, $args)
    );
}

Broadcast::channel('{wildcard}', fn () => false);  // hard default-deny
```

**Sanctum tokens:**
- Default TTL: 30 days.
- Prefix: `dera_pat_` (for GitHub secret-scanning compatibility).
- `last_used_at` updated on each authenticated request.
- Inactivity expiry: 90 days.
- Operator-visible "active sessions" UI with revoke per token.

**2FA recovery flow:**
- Backup codes generated at enrollment, shown ONCE, stored as `Hash::make($code)`.
- Each code single-use (deleted on consumption).
- Password reset does NOT mark the next session as 2FA-passed; user must complete 2FA after password reset.
- Disabling 2FA requires current TOTP code, not just password.
- Lost-2FA + lost-codes routed to `support@derablog.com` (manual identity check).

**Rate limits (`config/dera-rate-limits.php`):**
```php
return [
    'auth.login' => ['5/minute/ip', '10/hour/email'],
    'auth.password_reset' => ['3/hour/email'],
    'auth.register' => ['5/hour/ip'],
    'comments.submit' => ['10/hour/ip', '60/day/ip'],
    'forms.submit' => ['5/hour/ip'],
    'ai.call' => ['30/minute/user'],
    'newsletter.subscribe' => ['5/hour/ip'],
    'license.validate' => ['100/hour/key'],
    'webhook.lemon_squeezy' => ['100/minute/ip'],
];
```

**Phase + week:** Phase 1, Weeks 3–4.

---

## 3. Default-Deny Postures

| Surface | Old (implicit) | New (explicit) |
|---|---|---|
| Outbound HTTP | Implicit allow-all | `EgressGuard` denies private + cloud-metadata + non-https-for-vendor |
| Reverb channels | Whatever Laravel default | `routes/channels.php` requires explicit callback per channel; missing = 403 |
| Plugin install | Trust admin's judgment | Signed = auto-trust; unsigned = explicit per-install consent + `Community` badge |
| Page cache | Cache-everything | Skip on `Cookie`, `Authorization`, `Set-Cookie` in response, status ≠ 200 |
| AI calls | Implicit "user controls their key" | Hard cost ceiling, input cap, debounce, rate limit before any provider call |
| Media uploads | Trust file extension | Magic-byte sniff + re-encode raster + sanitize/reject SVG |
| `routes/channels.php` | Implicit | Wildcard `Broadcast::channel('{wildcard}', fn () => false)` at end |

---

## 4. Content Security Policy

**Header (set by `App\Http\Middleware\ContentSecurityPolicy`):**

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'nonce-{request-nonce}' 'strict-dynamic';
  style-src 'self' 'nonce-{request-nonce}';
  img-src 'self' data: https://{media-origin};
  font-src 'self';
  connect-src 'self' wss://{reverb-host};
  media-src 'self';
  object-src 'none';
  base-uri 'self';
  form-action 'self';
  frame-ancestors 'none';
  upgrade-insecure-requests;
  report-uri /security/csp-report
```

**Nonce strategy:**
- Per-request nonce generated in `RequestNonceMiddleware` (32-byte `random_bytes` → base64).
- Available in Blade as `@cspNonce`.
- Cached pages contain `{{NONCE_PLACEHOLDER}}` token; substituted at the response middleware after cache hit.

**CSP report endpoint:** `POST /security/csp-report` writes to a rate-limited log table; weekly report surfaces violations to admin.

---

## 5. Egress Allow-List Policy

Implemented in `EgressGuard` (Module A). Summary:

- **Always denied:** RFC1918, 169.254/16, 127/8, IPv6 link-local, `fc00::/7`, `fe80::/10`, `::1/128`.
- **Vendor calls** (`license`, `marketplace`, `telemetry`): HTTPS only, host-pinned, response-body size capped.
- **AI calls:** HTTPS only, host whitelist driven by `config/dera.php`, response cap 5 MB.
- **User-controlled URLs** (webhook, importer, link-check): public IPs only; importer additionally `Content-Type` allow-listed; link-check uses HEAD only.
- **DNS rebinding defense:** resolve → check → reconnect via `CURLOPT_RESOLVE` (no second resolution).

---

## 6. Trade-Off Ledger

| Trade-off | Cost | Decision | Mitigation |
|---|---|---|---|
| Strict-CSP nonce vs static page cache | Per-request nonce breaks naive caching | Cache with `{{NONCE_PLACEHOLDER}}`, substitute at response middleware | Cache hit ratio unchanged; CSP integrity preserved |
| EgressGuard vs developer DX | Local dev wants `host.docker.internal` | `APP_ENV=local` overlay (`dera-egress.dev.php`); production never loads it | Documented in CONTRIBUTING.md; integration test asserts |
| Signed plugins vs community velocity | Community devs cannot ship without signing coordination | Two-tier model: Verified (signed) vs Community (unsigned + consent + no auto-update) | Velocity preserved for community; brand-trust preserved for Verified |
| Hard AI cost ceiling vs admin frustration | Admin cannot raise beyond compile-time max | Default $50/mo covers normal use; env-overridable to $500; documented warning | Power users override at operator level (`.env` edit) |
| Hashed license keys vs support workflow | Support cannot read keys to help recovery | Customer portal shows key from encrypted-at-rest store + Lemon Squeezy receipt resend | Deterministic recovery flow without plaintext DB access |
| Single-key custody (Minisign) vs key-compromise blast radius | Key leak compromises all signed artifacts | YubiKey 5 hardware token; rotation procedure; bundle supports multiple key IDs | Manageable for solo ops; rotation tested in Phase 5 |
| Two-channel auth (session + Sanctum) vs simplicity | Two code paths | Single `User` model, single permissions table; channels detected in middleware | Sanctum is small surface, well-supported |
| Default-deny `routes/channels.php` vs feature velocity | Forgetting a callback breaks a feature | CI test enumerates registered channels and asserts each has a callback | Caught at PR time |

---

## 7. What We Are NOT Building (and Why)

| Deferred / rejected | Activation trigger | Why over-engineering for v1.0 |
|---|---|---|
| SOC 2 / ISO 27001 | First enterprise conversation, or 10K+ paying customers | $30K–$80K + 6–12 months of audit |
| HashiCorp Vault | First multi-region deployment | `.env` + filesystem permissions sufficient for one-server-per-customer |
| Distroless containers | First container-image scan request from a paying customer | `php:8.2-fpm-alpine` is small and well-maintained |
| Per-tenant DB / Redis isolation | DeraBlog Cloud beta (Year 3) | v1.0 is single-tenant per install |
| Third-party formal pen-test | First five paying enterprise customers, or first publicly-disclosed Critical | Phase-5 in-house pen-test of License Server + AIGateway + Plugin verification covers realistic threats |
| Cloudflare WAF tuning beyond defaults | Sustained scanner traffic > 10 req/s/IP | Default ruleset + app-layer rate limits handle script-kiddies |
| HackerOne / Bugcrowd bounty | First 100 active installs and a P0 surfaced via responsible disclosure | Triage capacity required (≈4 h/wk) that solo founder cannot service |
| SIEM / centralized log aggregation | First incident requiring multi-source forensics | Sentry + Pulse + activitylog sufficient for one host |
| HSM | License-server traffic > 1M validations/day | YubiKey + filesystem-isolated daemon is operationally simpler |
| mTLS PHP↔MySQL | DB instance not on same host | Same-host Unix socket is the v1.0 default |
| WebAuthn / passkeys | 5K+ MAU or recurring 2FA-recovery support tickets | TOTP + backup codes is the customer expectation |
| Formal GDPR DPIA + DPO | First explicit EU enterprise sale, or > 5K EU users | Privacy-by-design + DSAR + retention + processor list cover ops requirements |
| Anti-bot fingerprinting (commercial) | First scraping attack at scale, or paying customer demand | Cloudflare Turnstile + per-IP rate limits handle 99% of script-kiddie scrapers |

---

## 8. Phase Quality Gates

| Phase | CI auto-checks | Manual checks (founder runs once) | Pen-test priority |
|---|---|---|---|
| **1** | `composer audit`, `npm audit`, `gitleaks protect`, `pint --test`, `phpstan --level=6`, `pest --testsuite=security`, "no direct Guzzle outside EgressGuard" rule, "no direct provider SDK outside AIGateway" rule, channels.php has-callback test | Branch protection, signed-commit policy, GPG key reachable, `.env.example` only env file in repo, Docker prod runs as non-root | None — internal alpha only |
| **2** | All Phase-1 + XSS payload corpus on TipTap + comments + form submissions, rate-limit test suite, broken-link-checker SSRF test, Reverb channel-callback test | Soft-launch readiness: SECURITY.md PGP test (synthetic disclosure), Sentry integration, telemetry opt-in flow, activity log captures admin actions | Soft target: Comments + Forms (community probes) |
| **3** | All prior + Page Builder block-fuzzing corpus (≥30 payloads), Stripe webhook forgery test, DSAR export round-trip test | Walk through DSAR delete as fictional EU subscriber; verify all data removed except legally-retained billing | Page Builder XSS surface (high) |
| **4** | All prior + AI prompt-injection corpus (≥30 payloads), `AIGateway` cost-ceiling test, marketplace manifest tampering test, Lemon Squeezy webhook forgery test, license-key-hashing migration test, signed plugin verification test | Synthetic "trojan plugin" attempt against non-Verified marketplace listing; admin must see and consent to warning | License Server (Critical), AIGateway (Critical), Plugin signature verification (High) |
| **5** | All prior + fresh-install end-to-end test ("install + admin login + create post + add AI feature + survive payload corpus") | Re-run audit prompt against implemented system; record delta block | Final pen-test on top-3 surfaces; defer broader pen-test to first-paying-customer trigger |

---

## 9. Residual Risks (Accepted for v1.0)

1. **Solo founder reviews own code.** No second pair of eyes on Critical-severity changes. Mitigation: PHPStan rules + CI corpus + weekly synthetic security-review prompt. Accepted because: hiring a reviewer is cost-prohibitive at v1.0; rules > review for repetitive checks.

2. **Trojaned signed plugin remains possible** if YubiKey-backed root key is compromised. Mitigation: hardware-backed key, rotation procedure, multi-key-ID bundle. Accepted because: transparency-log alternatives are operational complexity a solo founder cannot maintain reliably.

3. **AI hard ceiling can be raised via `.env` edit** by determined-but-foolish operator. Reputational risk on "I edited my env and DeraBlog drained my key" remains. Accepted because: any in-app-only ceiling can be circumvented by a determined operator anyway; locating override at operator level (not admin level) is the best honest split.

4. **Cache poisoning across locales** if `getPreferredLanguage` parsing has an edge case. Mitigation: explicit allow-list in `composeCacheKey`. Accepted because: allow-list is bounded and unit-tested; future locales added via code.

5. **First-time-installed Community plugin still grants PHP RCE on admin click.** "I trust this source" consent is real but not absolute. Accepted because: anything stronger (refusing to load unsigned PHP) kills the community ecosystem entirely; documenting trust state in-UI and never auto-updating Community plugins limits blast radius to deliberate admin action.

These residuals are the price of shipping a community-extensible, BYOK-friendly, privacy-respecting, single-founder CMS in 26 weeks. They are visible, named, and bounded.

---

## 10. Mapping: Audit Findings → Closure

| Finding | Severity | Module(s) | Phase delivered | Closure mechanism |
|---|---|---|---|---|
| F-1 SSRF — Automation Webhook | High | EgressGuard | Phase 1 W2 (defense), Phase 4 W19 (feature using it) | `webhook` profile blocks RFC1918 + cloud metadata |
| F-2 SSRF — Importer + Link Checker | High | EgressGuard | Phase 1 W5 (Importer), Phase 2 W7 (Link Checker) | `import` and `link_check` profiles |
| F-3 Plugin RCE | Critical | ArtifactTrust | Stub Phase 1 W6, full Phase 4 W22 | Signed packages + two-tier trust + admin consent for Community |
| F-4 AI cost amplification | High | AIGateway | Phase 4 W19 | Input cap + debounce + hard ceiling enforced before provider call |
| F-5 Page Builder XSS | High | ContentSanitizer | Phase 1 W3 (baseline), Phase 3 W13 (Page Builder profile) | Per-block-type purifier profile; HTML block admin-only |
| F-6 Lemon Squeezy webhook forgery | High | ArtifactTrust | License Server build (Q3 2026) | HMAC-SHA256 verify + 5-min replay window |
| F-7 License-key plaintext storage | High | ArtifactTrust | License Server build (Q3 2026) | `hash_license_key()` at issuance; plaintext shown once |
| F-8 SVG / polyglot upload | Medium | ContentSanitizer | Phase 1 W5 | `sanitizeSvg()` or reject + raster re-encode + magic-byte check + isolated media origin |
| F-9 Sanctum lifecycle | Medium | AuthEdge | Phase 1 W3 | 30d TTL + `last_used_at` + 90d inactivity expiry + `dera_pat_` prefix |
| F-10 Reverb channel auth | Medium | AuthEdge | Phase 1 W3 (default-deny stub), Phase 2 W12 (per-feature) | `routes/channels.php` wildcard default-deny + per-channel callbacks |
| F-11 Cache-key poisoning | Medium | AuthEdge | Phase 1 W4 | `composeCacheKey()` formula + skip on Cookie/Authorization |
| F-12 2FA recovery back door | Medium | AuthEdge | Phase 1 W3 | Backup codes once, hashed, single-use; password reset does not bypass 2FA |
| F-13 Theme update injection | High | ArtifactTrust | Phase 4 W22 | Signed theme ZIPs verified before activation |
| F-14 Newsletter subscription bombing | Medium | AuthEdge | Phase 2 W11 | Turnstile + per-IP rate limit (5/h) + double opt-in via signed token |

---

## 11. References

- OWASP ASVS 4.0.3 — used as the underlying control taxonomy.
- SANS CWE Top 25 (2024).
- Laravel Security Best Practices (laravel.com/docs/11.x/security).
- Mozilla Observatory grading (target: A+ before Full Launch).
- Strict-CSP guidance — Google Web.dev `strict-csp`.
- Minisign documentation (https://jedisct1.github.io/minisign/).
