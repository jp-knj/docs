# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is the official Astro documentation website (https://docs.astro.build/), built with Astro and Starlight. The site supports 14 languages with automated translation tracking via Lunaria.

## Essential Commands

### Development
```bash
# Start development server
pnpm dev

# Build for production
pnpm build

# Preview production build
pnpm preview

# Type check TypeScript
pnpm check
```

### Testing & Validation
```bash
# Run all linting checks
pnpm lint

# Check for broken links (important before PR)
pnpm lint:linkcheck

# Validate URL slugs
pnpm lint:slugcheck

# Lint JavaScript/TypeScript
pnpm lint:eslint

# Lint Japanese translations with textlint
pnpm lint:ja

# Format code
pnpm format

# Run specific tests
pnpm test:links  # Test internal links
pnpm test:unit   # Run unit tests
```

### Documentation Generation
```bash
# Generate TypeScript config reference
pnpm docgen tsconfig

# Build Lunaria translation dashboard
pnpm lunaria:build
```

## Architecture & Key Concepts

### Content Structure
- **Source content**: `/src/content/docs/en/` (English is the source language)
- **Translations**: `/src/content/docs/[lang]/` where lang = ja, fr, es, de, ko, zh-cn, etc.
- **Content hierarchy**:
  - `basics/` - Getting started guides
  - `concepts/` - Core Astro concepts
  - `guides/` - Feature-specific guides (backend, cms, deploy, integrations-guide, internationalization)
  - `recipes/` - Common patterns and solutions
  - `reference/` - API and configuration reference
  - `tutorial/` - Step-by-step tutorial

### Translation Workflow
1. English content is the source of truth
2. Each translated file must have `i18nReady: true` in frontmatter when complete
3. Lunaria tracks translation status automatically
4. PRs update translation status unless labeled "en-only"

### Custom Components & Overrides
- Starlight theme overrides: `/src/components/starlight/`
- Tutorial components: `/src/components/tutorial/`
- Custom UI components extend Starlight's defaults

### Key Configuration Files
- `astro.config.ts` - Main Astro configuration with Starlight setup
- `config/locales.ts` - Language configuration and labels
- `config/sidebar.ts` - Documentation navigation structure
- `.textlintrc.ja.json` - Japanese documentation style rules
- `lunaria.config.json` - Translation tracking configuration

### Japanese Translation Guidelines
When translating to Japanese:
1. Use textlint configuration at `.textlintrc.ja.json` and `.textlintrc-ja-docs`
2. Follow terminology in `/src/content/i18n/ja.yml`
3. Key terms (first occurrence): Include English in parentheses, e.g., "サービスワーカー (Service Worker)"
4. Convert internal links: `/en/...` → `/ja/...`
5. Run `pnpm lint:ja` to validate Japanese text quality

### Development Tips
- The site uses Astro's content collections - modify schemas in `/src/content/config.ts`
- ExpressiveCode provides collapsible code blocks via `collapse` and `mark` directives
- OG images are auto-generated via `/src/pages/open-graph/[...path].ts`
- Search is powered by Algolia DocSearch
- Build performance is optimized with split API routes and selective prerenders