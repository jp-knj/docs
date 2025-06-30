# Astro ドキュメント翻訳ワークフロー

## 概要
このドキュメントは、Astro ドキュメントの実務導線ガイド（backend / cms / deploy / integrations-guide / internationalization など）を英語から日本語へ翻訳する際のワークフローを説明します。

## 翻訳パイプライン

### 1. コード/CLI検証（Claude）
- すべてのコードブロック、シェルコマンド、設定スニペットをスキャン
- 問題がある場合（タイポ、古いフラグ、不足インポート）はインラインで修正し、`<!-- ✅ fixed by Claude -->` を追加
- 変更が不要な場合は `<!-- ✔ verified by Claude -->` を追加

### 2. 翻訳（Gemini）
- 検証済みの英語MDXをGeminiに送信して日本語翻訳
- GeminiがフルのJapanese MDXを返す

### 3. スタイルリンティング（textlint）
- 翻訳されたMDXを `textlint --config .textlintrc-ja-docs --fix` で実行
- 自動修正可能なものは修正し、警告が残る場合は以下を追加：
  `<!-- textlint-remaining: <list messages> -->`

## 翻訳ルール

### 必須ルール
1. フロントマターをすべて保持し、翻訳完了時に `i18nReady: true` を設定
2. コードブロック、コメント、ファイルパスは元のままを維持（ステップ1で修正されない限り）
3. 内部リンクをすべて `/en/...` → `/ja/...` に変換
4. Astro日本語スタイルガイドに従う：
   - 用語統一辞書に従う（例：Supabase → スーパーベース）
   - 半角英数字・全角記号の混在を避ける
   - 1文60字前後で改行
5. 初出時は英語を括弧内に保持：「サービスワーカー (Service Worker)」

## 実行例

### 1. 単一ファイルの翻訳（appwriteio.mdx）

```bash
# 英語版ファイルを読み込み
src/content/docs/en/guides/backend/appwriteio.mdx

# 日本語版として保存
src/content/docs/ja/guides/backend/appwriteio.mdx
```

翻訳結果：
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

### 2. 複数ファイルの一括翻訳

現在、以下のファイルがja/guides/backend/に存在しません：
- google-firebase.mdx
- neon.mdx
- sentry.mdx
- supabase.mdx
- turso.mdx

## textlint実行方法

```bash
# 個別ファイルのチェック
pnpm textlint --config .textlintrc.ja.json src/content/docs/ja/guides/backend/appwriteio.mdx

# 全日本語ドキュメントのチェック
pnpm lint:ja
```

## 翻訳ステータス

| ファイル名 | ステータス | 備考 |
|-----------|----------|------|
| appwriteio.mdx | ✅ 完了 | textlint検証済み |
| google-firebase.mdx | 🔄 進行中 | - |
| neon.mdx | ⏳ 保留 | - |
| sentry.mdx | ⏳ 保留 | - |
| supabase.mdx | ⏳ 保留 | - |
| turso.mdx | ⏳ 保留 | - |

## 注意事項

- 外部リンクの検証は不要（翻訳時には行わない）
- コードブロック内のコメントは英語のまま維持
- 用語統一辞書（/src/content/i18n/ja.yml）を常に参照
- PRを作成する前に必ず `pnpm lint:ja` を実行して検証