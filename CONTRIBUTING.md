# Contributing to DeraBlog

First — thank you for considering contributing! DeraBlog is built in public and welcomes contributions of all kinds: code, documentation, themes, plugins, translations, design, and ideas.

---

## Code of Conduct

Participation in this project is governed by our [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you agree to uphold its terms.

---

## Project Status

DeraBlog is currently in **pre-alpha development**. The repository is public for transparency and to build community early, but the codebase is under heavy iteration. The first stable release (v1.0) is targeted for Week 24 of development.

During pre-alpha, breaking changes are expected. Pull requests are welcome but please open an issue first to discuss substantial changes.

---

## How to Contribute

### 1. Reporting Bugs

- Check [existing issues](https://github.com/derabia/DeraBlog/issues) first.
- Open a new issue with:
  - **Clear title** describing the problem.
  - **Steps to reproduce.**
  - **Expected vs. actual behavior.**
  - **Environment** (PHP version, OS, browser if relevant).
  - **Screenshots or logs** when applicable.

### 2. Suggesting Features

- Check the [PLAN.md](PLAN.md) — your idea may already be on the roadmap.
- Open a "Feature Request" issue with:
  - **Problem statement** — what user need does this address?
  - **Proposed solution.**
  - **Alternatives considered.**
  - **Whether you'd be willing to implement it.**

### 3. Submitting Pull Requests

- **Fork** the repository.
- **Create a branch** from `main`: `git checkout -b feature/my-feature`.
- **Follow the code style** (Laravel Pint for PHP, Prettier for JS/CSS).
- **Write tests** for new functionality (Pest is preferred over PHPUnit).
- **Update documentation** if your change affects user-facing behavior.
- **Use Conventional Commits** for commit messages:
  - `feat:` new feature
  - `fix:` bug fix
  - `docs:` documentation
  - `style:` formatting (no code change)
  - `refactor:` code change that neither fixes a bug nor adds a feature
  - `test:` adding or updating tests
  - `chore:` maintenance tasks
- **Open a pull request** against `main` with:
  - Clear title (using Conventional Commit format).
  - Description of the change and why it's needed.
  - Reference to related issues (`Closes #123`).
  - Screenshots/recordings for UI changes.

### 4. Building Themes

Themes live in `themes/{slug}/`. The full theme development guide will be published with v1.0 at [docs.derablog.com](https://docs.derablog.com). For now, see `ARCHITECTURE.md` Section 5.

Quick start (once the CLI is implemented):
```bash
php artisan dera:theme:make my-theme
```

### 5. Building Plugins

Plugins live in `plugins/{slug}/`. The full plugin development guide will be published with v1.0 at [docs.derablog.com](https://docs.derablog.com). For now, see `ARCHITECTURE.md` Section 7.

Quick start (once the CLI is implemented):
```bash
php artisan dera:plugin:make my-plugin
```

### 6. Translations

DeraBlog supports many languages. Translation files live in:
- `lang/{locale}/` for the Core
- `themes/{theme}/lang/{locale}/` for themes
- `plugins/{plugin}/resources/lang/{locale}/` for plugins

To add a new locale, copy the `en/` folder to `{locale}/` and translate the strings.

---

## Development Setup

### Prerequisites

- PHP 8.2+
- Composer 2.x
- Node.js 20+ (for asset building)
- MySQL 8+ or MariaDB 10.6+
- Redis 6+

### Local Setup

```bash
git clone https://github.com/derabia/DeraBlog.git
cd DeraBlog
composer install
npm install
cp .env.example .env
php artisan key:generate
php artisan migrate --seed
npm run dev
php artisan serve
```

Or with Docker:
```bash
docker compose up -d
docker compose exec app php artisan migrate --seed
```

### Running Tests

```bash
php artisan test
# or
./vendor/bin/pest
```

### Code Style

```bash
./vendor/bin/pint           # Format PHP
npm run lint                 # Lint JS/CSS
```

---

## Plugin & Theme Marketplace

If you're interested in selling plugins or themes for DeraBlog through the future official marketplace (Year 2), reach out to: marketplace@derablog.com.

The Verified Developer Program ($99/year, Year 2+) gives developers:
- Listing in the official marketplace
- 70–80% revenue share (20–30% platform commission)
- Featured placement opportunities
- Direct support channel

---

## Financial Support

DeraBlog is bootstrapped and intentionally not seeking VC funding. If you'd like to support development:

- ⭐ **Star the repository** — visibility is the most valuable currency for an open-source project.
- 🧪 **Use it and report issues** — real-world testing is invaluable.
- 🎨 **Contribute themes/plugins** — the ecosystem makes the product.
- 💬 **Spread the word** — Twitter/X, blog posts, community discussions.
- 💰 **Buy a Premium Theme** when they launch — direct financial support.

---

## License

By contributing to DeraBlog Core, you agree that your contributions will be licensed under the [MIT License](LICENSE).

For premium themes and add-ons, contributors should arrange a separate agreement with Derabia.com (typically a revenue-sharing or work-for-hire arrangement).

---

## Questions?

- Open an issue.
- Email: hello@derablog.com
- Twitter/X: [@derablog](https://twitter.com/derablog) *(launching soon)*
