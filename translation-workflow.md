# Astro Documentation Translation Workflow

## Overview
This document explains the workflow for translating Astro documentation practical guides (cms / deploy / integrations-guide / internationalization, etc.) from English to Japanese.

## Translation Pipeline

### 1. Code/CLI Verification (Claude)
- Scan all code blocks, shell commands, and configuration snippets
- If issues are found (typos, outdated flags, missing imports), fix inline and add `<!-- ✅ fixed by Claude -->`
- If no changes needed, add `<!-- ✔ verified by Claude -->`

### 2. Translation (Gemini)
- Send verified English MDX to Gemini for Japanese translation
- Gemini returns full Japanese MDX

### 3. Style Linting (textlint)
- Run translated MDX through `textlint --config .textlintrc-ja-docs --fix`
- Auto-fix what's possible, and if warnings remain, add:
  `<!-- textlint-remaining: <list messages> -->`

## Translation Rules

### Required Rules
1. Preserve all frontmatter and set `i18nReady: true` when translation is complete
2. Keep code blocks, comments, and file paths as-is (unless fixed in step 1)
3. Convert all internal links from `/en/...` → `/ja/...`
4. Follow Astro Japanese Style Guide:
   - Follow terminology dictionary (e.g., Supabase → スーパーベース)
   - Avoid mixing half-width alphanumeric and full-width symbols
   - Line breaks around 60 characters per sentence
5. For first occurrence, keep English in parentheses: "サービスワーカー (Service Worker)"

## Execution Examples

### 1. Single File Translation (appwriteio.mdx)

```bash
# Read English source file
src/content/docs/en/guides/backend/appwriteio.mdx

# Save as Japanese version
src/content/docs/ja/guides/backend/appwriteio.mdx
```

Translation result:
```mdx
---
title: Appwrite & Astro
description: Appwrite でプロジェクトにバックエンドを追加する
sidebar:
  label: Appwrite
type: backend
service: Appwrite
stub: true
i18nReady: true
---

[Appwrite](https://appwrite.io/) は、認証とアカウント管理、ユーザー設定、データベースとストレージの永続化、クラウド関数、ローカライゼーション、画像操作、その他のサーバーサイドユーティリティを提供するセルフホスト型のバックエンド・アズ・ア・サービス (Backend-as-a-Service) プラットフォームです。

## 公式リソース
- [Astro 向け Appwrite デモ](https://github.com/appwrite/demos-for-astro)
```

### 2. Batch Translation of Multiple Files

Currently, the following files do not exist in ja/guides/backend/:
- google-firebase.mdx
- neon.mdx
- sentry.mdx
- supabase.mdx
- turso.mdx

## textlint Execution

```bash
# Check individual file
pnpm textlint --config .textlintrc.ja.json src/content/docs/ja/guides/backend/appwriteio.mdx

# Check all Japanese documentation
pnpm lint:ja
```

## Translation Status

| File Name | Status | Notes |
|-----------|--------|-------|
| appwriteio.mdx | ✅ Complete | textlint verified |
| google-firebase.mdx | 🔄 In Progress | - |
| neon.mdx | ⏳ Pending | - |
| sentry.mdx | ⏳ Pending | - |
| supabase.mdx | ⏳ Pending | - |
| turso.mdx | ⏳ Pending | - |

## Important Notes

- External link verification is not required (not done during translation)
- Comments in code blocks should remain in English
- Always refer to terminology dictionary (/src/content/i18n/ja.yml)
- Always run `pnpm lint:ja` for verification before creating PR
