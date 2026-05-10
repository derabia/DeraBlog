# Security Policy

We take the security of DeraBlog seriously. This document explains our policy for reporting and handling security vulnerabilities in DeraBlog Core and the official premium themes.

---

## Supported Versions

| Version    | Status               | Security Updates |
|------------|----------------------|------------------|
| `0.x` (pre-alpha) | Active development   | ⚠️ Best-effort. **Not for production use.** |
| `1.x`      | Not yet released     | Will receive security updates for 12 months after the next minor release. |

DeraBlog is currently in **pre-alpha**. Do not run it on a production site that handles real user data. The first stable release (`v1.0.0`) is planned at the end of the 24-week MVP timeline (see `PLAN.md`).

---

## Reporting a Vulnerability

**Please do not open a public GitHub issue for security vulnerabilities.** Public disclosure before a fix is in place puts every user at risk.

### Preferred channel — Private email

Send a detailed report to **`security@derablog.com`**.

If you wish to encrypt your report, request our PGP public key in the first email and we will reply with the current key fingerprint. (PGP key publication is planned alongside the v0.1 release.)

### Alternative channel — GitHub Security Advisory

You can also use [GitHub's private vulnerability reporting](https://github.com/derabia/DeraBlog/security/advisories/new) on this repository. This is the most convenient option if you already have a GitHub account.

### What to include

A good security report contains:

1. **Affected component** — Core CMS, a specific premium theme, the License Server, or another DeraBlog repository.
2. **Affected version(s)** — commit SHA or release tag.
3. **Vulnerability class** — XSS, SQLi, RCE, IDOR, SSRF, auth bypass, privilege escalation, etc.
4. **Impact** — what an attacker can do, who is affected, and any preconditions.
5. **Reproduction steps** — minimal proof-of-concept (code, request, payload).
6. **Suggested mitigation** — optional but appreciated.

---

## Our Commitment

When you report a vulnerability privately, we commit to the following timeline:

| Action                                                                    | Target     |
|---------------------------------------------------------------------------|------------|
| Acknowledge receipt of your report                                        | ≤ 72 hours |
| Confirm whether the issue is reproducible / classified as a vulnerability | ≤ 7 days   |
| Initial fix or mitigation deployed for **Critical** issues                | ≤ 7 days   |
| Initial fix or mitigation deployed for **High** issues                    | ≤ 30 days  |
| Initial fix or mitigation deployed for **Medium / Low** issues            | ≤ 90 days  |
| Public disclosure (CVE / advisory) after coordinated fix                  | mutually agreed |

We follow a **coordinated disclosure** model: we will not disclose your report publicly until a fix is available, and we will credit you in the security advisory and `CHANGELOG.md` (unless you prefer to remain anonymous).

---

## Scope

### In scope

- The DeraBlog Core CMS (`derabia/DeraBlog`).
- The DeraBlog License Server (`derabia/derablog-license-server`).
- Official premium themes published under the `derabia` GitHub organization.
- Default plugins shipped with the Core repository.
- The DeraBlog website and documentation site if they are exposing user data or authenticated sessions.

### Out of scope

- Third-party plugins and themes not published by `derabia`. Report those to their maintainers.
- Self-hosted installations that have been modified by the operator. We can advise but cannot patch your fork.
- Denial-of-service attacks that require unrealistic resources or cooperation from the victim.
- Issues that require already-compromised admin credentials with no privilege escalation.
- Findings from automated scanners without a working proof-of-concept.

---

## Premium Themes

Vulnerabilities in **official premium themes** (sold under the Commercial EULA — see `LICENSE-COMMERCIAL.md`) follow the same disclosure process as the Core CMS. Customers with an active license also have access to the priority support channel via the customer portal at [derablog.com](https://derablog.com).

---

## Hall of Fame

Researchers who report valid vulnerabilities will be credited in:

- The corresponding entry in `CHANGELOG.md`.
- The published GitHub Security Advisory.
- A future "Security Acknowledgements" page on the project website.

If you would like to remain anonymous or prefer a different attribution, please mention this in your report.

---

## No Bug Bounty (Yet)

DeraBlog is a small open-source project and does not currently offer a paid bug bounty. We deeply appreciate every good-faith security report and will recognize researchers publicly. A formal bounty program may be introduced in the future as the project matures and revenue stabilizes.

---

## Questions

For non-vulnerability security questions (best practices, hardening, configuration), please open a regular discussion on [GitHub Discussions](https://github.com/derabia/DeraBlog/discussions) or refer to the documentation site once it is published.

Thank you for helping keep DeraBlog and its users safe.
