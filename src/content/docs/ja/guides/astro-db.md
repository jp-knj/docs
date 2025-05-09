---
title: 'Astro DB'
description: Astro専用に設計された完全管理型SQLデータベース、Astro DBの使い方を学びましょう。
githubIntegrationURL: '[https://github.com/withastro/astro/tree/main/packages/db/](https://github.com/withastro/astro/tree/main/packages/db/)'
i18nReady: true
---

import { FileTree } from '@astrojs/starlight/components';
import PackageManagerTabs from '~~/components/tabs/PackageManagerTabs.astro';
import ReadMore from '~~/components/ReadMore.astro';
import Since from '\~/components/Since.astro';
import { Steps } from '@astrojs/starlight/components';

Astro DBは、Astroエコシステム向けに設計された完全管理型のSQLデータベースです。Astro内でローカル開発を行い、libSQL互換のデータベースへデプロイできます。

Astro DBは、データの設定、開発、クエリ実行をまとめて行える完全なソリューションです。`astro dev`を実行すると、`.astro/content.db`内にローカルデータベースが作成され、Dockerやネットワーク接続なしでデータを管理できます。

## インストール

組み込みの`astro add`コマンドを使って[`@astrojs/db`インテグレーション](/ja/guides/integrations-guide/db/)をインストールします。

<PackageManagerTabs>
  <Fragment slot="npm">
  ```sh
  npx astro add db
  ```
  </Fragment>
  <Fragment slot="pnpm">
  ```sh
  pnpm astro add db
  ```
  </Fragment>
  <Fragment slot="yarn">
  ```sh
  yarn astro add db
  ```
  </Fragment>
</PackageManagerTabs>

## データベースを定義する

`astro add`コマンドで`@astrojs/db`をインストールすると、プロジェクト内に`db/config.ts`ファイルが自動作成され、ここでデータベーステーブルを定義します。

```ts title="db/config.ts"
import { defineDb } from 'astro:db';

export default defineDb({
  tables: { },
})
```

### テーブル

Astro DBでは、データはSQLテーブルを使って保存されます。テーブルはデータを行と列で構造化し、各列は行の値の型を制約します。

既存のlibSQLデータベース内のデータ、または新規データベースで収集するデータの構造を`db/config.ts`ファイル内で定義します。これにより、Astroはそのテーブルにクエリを投げるためのTypeScriptインターフェースを生成します。結果として、プロパティの自動補完や型チェック付きでデータへアクセスでき、完全なTypeScriptサポートが得られます。

データベーステーブルを設定するには、`astro:db`から`defineTable()`と`column`ユーティリティをインポートして使用します。その後、テーブルの名前（大文字小文字を区別）と各列に入るデータの型を定義します。

以下は、`author`と`body`という必須のテキスト列を持つ`Comment`テーブルを設定し、それを`defineDb()`のエクスポートでプロジェクト内で利用可能にする例です。

```ts title="db/config.ts" "Comment"
import { defineDb, defineTable, column } from 'astro:db';

const Comment = defineTable({
  columns: {
    author: column.text(),
    body: column.text(),
  }
})

export default defineDb({
  tables: { Comment },
})
```

<ReadMore>[テーブル設定リファレンス](/ja/guides/integrations-guide/db/#table-configuration-reference)を参照してください。</ReadMore>

### 列（カラム）

Astro DBは以下の列タイプをサポートしています。

```ts title="db/config.ts" "column.text()" "column.number()" "column.boolean()" "column.date()" "column.json()"
import { defineTable, column } from 'astro:db';

const Comment = defineTable({
  columns: {
    // テキスト文字列。
    author: column.text(),
    // 整数値。
    likes: column.number(),
    // 真偽値。
    flagged: column.boolean(),
    // JavaScriptのDateオブジェクトとしてクエリされる日時値。
    published: column.date(),
    // 型指定されていないJSONオブジェクト。
    metadata: column.json(),
  }
});
```

<ReadMore>[テーブル列リファレンス](/ja/guides/integrations-guide/db/#table-configuration-reference)も参照してください。</ReadMore>

### テーブル参照

テーブル間のリレーションは、データベース設計における一般的なパターンです。たとえば、`Blog`テーブルは`Comment`、`Author`、`Category`といった他のテーブルと密接に関連する場合があります。

テーブル間の関係は**参照カラム**を使って定義し、データベーススキーマ内に保存できます。リレーションを確立するには、以下が必要です。

- 参照される側のテーブルに**識別子カラム**（通常は`primaryKey`プロパティを持つ`id`カラム）があること。
- 基本テーブルに**参照する`id`を保存するカラム**を設置すること。このカラムは`references`プロパティを使用して関係を確立します。

以下は、`Comment`テーブルの`authorId`カラムが`Author`テーブルの`id`カラムを参照している例です。

```ts title="db/config.ts" {3, 10}
const Author = defineTable({
  columns: {
    id: column.number({ primaryKey: true }),
    name: column.text(),
  }
});

const Comment = defineTable({
  columns: {
    authorId: column.number({ references: () => Author.columns.id }),
    body: column.text(),
  }
});
```

## 開発用データベースのシード

開発環境では、AstroはDB設定を使用してスキーマに基づいたローカル型を生成します。これらの型は開発サーバー起動時に毎回シードファイルから新しく生成され、型安全性とオートコンプリート付きでデータのクエリや操作ができます。

開発中に[リモートデータベースへ接続](#リモートデータベースへの接続)しない限り、本番データへアクセスすることはできません。これによりデータを保護しつつ、型安全性のある動作データベースを用いたテスト・開発が可能になります。

テストやデバッグ用の開発データをAstroプロジェクトにシードするには、`db/seed.ts`ファイルを作成します。`astro:db`から定義済みの`db`オブジェクトやテーブルをインポートし、各テーブルに初期データを`insert`します。この開発用データは、データベーススキーマおよび本番データの形式と一致させる必要があります。

以下の例では、`Comment`テーブルと`Author`テーブルにそれぞれ2行の開発用データを定義しています。

```ts title="db/seed.ts"
import { db, Comment, Author } from 'astro:db';

export default async function() {
  await db.insert(Author).values([
    { id: 1, name: "Kasim" },
    { id: 2, name: "Mina" },
  ]);

  await db.insert(Comment).values([
    { authorId: 1, body: 'Hope you like Astro DB!' },
    { authorId: 2, body: 'Enjoy!'},
  ])
}
```

開発サーバーはこのファイルの変更を検知すると自動的にデータベースを再起動し、型を再生成し、`seed.ts`から新たに開発データをシードします。

## 本番用のlibSQLデータベースに接続する

Astro DBは、ローカルのlibSQLデータベース、またはlibSQLリモートプロトコルを公開しているマネージド・セルフホスト型サーバーのどちらにも接続できます。

Astro DBをlibSQLデータベースに接続するには、データベースプロバイダーから取得した以下の環境変数を設定します。

- `ASTRO_DB_REMOTE_URL`: ローカルまたはリモートlibSQL DBの接続URL。このURLには、同期や暗号化などの[URL設定オプション](#リモートurl設定オプション)をパラメータとして含めることができます。
- `ASTRO_DB_APP_TOKEN`: libSQLサーバーの認証トークン。これはリモートデータベースに必要であり、[ローカルDB（ファイルまたはメモリ内）](#urlスキームとホスト)には不要です。

サービスによってはCLIやWeb UIを使ってこれらの値を取得できます。以下では、例としてTursoを使用し、この値を設定する手順を示しますが、他のプロバイダーも自由に利用できます。

### Tursoの始め方

Tursoは、Astro DBを支えるオープンソースのSQLiteフォーク[libSQL](https://github.com/tursodatabase/libsql)の開発元です。完全管理型のlibSQLデータベースプラットフォームを提供し、Astroと完全互換です。

以下の手順では、Turso CLIのインストール、ログイン（またはサインアップ）、新しいデータベースの作成、必要な環境変数の取得、スキーマのリモートデータベースへのプッシュ方法を案内します。

<Steps>

1. [Turso CLI](https://docs.turso.tech/cli/installation)をインストールします。

2. Tursoに[ログインまたはサインアップ](https://docs.turso.tech/cli/authentication)します。

3. 新しいデータベースを作成します。例として、データベース名を`andromeda`とします。

   ```sh "andromeda"
   turso db create andromeda
   ```

4. `show`コマンドを実行して、新しく作成したデータベースの情報を確認します。

   ```sh "andromeda"
   turso db show andromeda
   ```

   出力される`URL`値をコピーし、`ASTRO_DB_REMOTE_URL`に設定します。

   ```dotenv title=".env" "libsql://andromeda-houston.turso.io"
   ASTRO_DB_REMOTE_URL=libsql://andromeda-houston.turso.io
   ```

5. データベースへのリクエストを認証する新しいトークンを作成します。

   ```sh "andromeda"
   turso db tokens create andromeda
   ```

   コマンド出力をコピーし、`ASTRO_DB_APP_TOKEN`に設定します。

   ```dotenv title=".env" add={2} "eyJhbGciOiJF...3ahJpTkKDw"
   ASTRO_DB_REMOTE_URL=libsql://andromeda-houston.turso.io
   ASTRO_DB_APP_TOKEN=eyJhbGciOiJF...3ahJpTkKDw
   ```

6. DBスキーマとメタデータを新しいTursoデータベースにプッシュします。

   ```sh
   astro db push --remote
   ```

7. 接続が完了しました！少し休憩しましょう。👾

   ```sh
   turso relax
   ```

</Steps>

Tursoの詳細機能については、[Tursoドキュメント](https://docs.turso.tech)を参照してください。

## リモートデータベースへの接続

Astro DBでは、ローカルおよびリモートのデータベースへの接続が可能です。デフォルトでは、Astroは`dev`および`build`コマンド用にローカルデータベースファイルを使用し、毎回テーブルを再作成して開発用のシードデータを挿入します。

ホスティングされたリモートデータベースに接続するには、`--remote`フラグを使用します。このフラグにより、リモートデータベースへの読み取りおよび書き込みアクセスが有効化され、本番環境で[ユーザーデータを受け取り、永続化](#インサート)できます。

ビルドコマンドに`--remote`フラグを追加して構成します。

```json title="package.json" "--remote"
{
  "scripts": {
    "build": "astro build --remote"
  }
}
```

また、コマンドラインで直接フラグを使用することもできます。

```bash
# リモート接続でビルド
astro build --remote

# リモート接続で開発
astro dev --remote
```

`--remote`フラグは、ローカルのビルド中およびサーバー上の両方でリモートDBへの接続を使用します。ローカル開発環境とデプロイメントプラットフォームの両方に必要な環境変数を必ず設定してください。

Astro DBプロジェクトをデプロイする際は、デプロイメントプラットフォームのビルドコマンドを`npm run build`（または使用するパッケージマネージャーの同等のコマンド）に設定し、`package.json`内で構成済みの`--remote`フラグを利用できるようにしてください。

### リモートURL設定オプション

`ASTRO_DB_REMOTE_URL`環境変数は、データベースの場所および同期や暗号化などの他のオプションを構成します。

#### URLスキームとホスト

libSQLは、リモートサーバー用のトランスポートプロトコルとしてHTTPおよびWebSocketの両方をサポートしています。また、ローカルファイルやインメモリDBの使用も可能です。これらは接続URL内の以下のスキームで構成できます。

- `memory:` → インメモリDBを使用（この場合ホストは空）。
- `file:` → ローカルファイルを使用（ホストはファイルへのパス例：`file:path/to/file.db`）。
- `libsql:` → ライブラリが推奨するプロトコルでリモートサーバーを使用（バージョンによって異なる場合あり）。ホストはサーバーのアドレス（例：`libsql://your.server.io`）。
- `http:` → HTTP経由でリモートサーバーを使用。`https:`を使用するとセキュアな接続が可能。ホストは`libsql:`と同じ。
- `ws:` → WebSocket経由でリモートサーバーを使用。`wss:`を使用するとセキュアな接続が可能。ホストは`libsql:`と同じ。

libSQL接続の詳細（例：暗号化キー、レプリケーション、同期間隔）は、リモート接続URL内のクエリパラメータとして構成できます。

たとえば、暗号化されたローカルファイルをlibSQLサーバーの埋め込みレプリカとして使用する場合、以下の環境変数を設定します。

```dotenv title=".env"
ASTRO_DB_REMOTE_URL=file://local-copy.db?encryptionKey=your-encryption-key&syncInterval=60&syncUrl=libsql%3A%2F%2Fyour.server.io
ASTRO_DB_APP_TOKEN=token-to-your-remote-url
```

#### `encryptionKey`

libSQLは暗号化データベースをネイティブにサポートしています。この検索パラメータを渡すことで、指定したキーを使用して暗号化が有効になります。

```dotenv title=".env"
ASTRO_DB_REMOTE_URL=file:path/to/file.db?encryptionKey=your-encryption-key
```

#### `syncUrl`

埋め込みレプリカは、libSQLクライアントの機能で、ローカルファイルまたはインメモリ上に完全に同期されたデータベースのコピーを作成し、超高速読み取りを実現します。書き込みは`syncUrl`で定義されたリモートデータベースに送信され、ローカルコピーと同期されます。

このプロパティを使用して、別のデータベースの埋め込みレプリカに変換するための接続URLを渡します。使用できるのは`file:`および`memory:`スキームのみです。パラメータはURLエンコードする必要があります。

たとえば、`libsql://your.server.io`上のデータベースのインメモリ埋め込みレプリカを作成する場合、次のように接続URLを設定します。

```dotenv title=".env"
ASTRO_DB_REMOTE_URL=memory:?syncUrl=libsql%3A%2F%2Fyour.server.io
```

#### `syncInterval`

埋め込みレプリカの同期間隔（秒単位）。デフォルトでは起動時と書き込み後のみ同期されます。

このプロパティは`syncUrl`が設定されている場合のみ使用されます。たとえば、1分ごとに同期するインメモリ埋め込みレプリカを設定するには、次の環境変数を指定します。

```dotenv title=".env"
ASTRO_DB_REMOTE_URL=memory:?syncUrl=libsql%3A%2F%2Fyour.server.io&syncInterval=60
```

## Query your database

You can query your database from any [Astro page](/en/basics/astro-pages/#astro-pages), [endpoint](/en/guides/endpoints/), or [action](/en/guides/actions/) in your project using the provided `db` ORM and query builder.

### Drizzle ORM

```ts
import { db } from 'astro:db';
```

Astro DB includes a built-in [Drizzle ORM](https://orm.drizzle.team/) client. There is no setup or manual configuration required to use the client. The Astro DB `db` client is automatically configured to communicate with your database (local or remote) when you run Astro. It uses your exact database schema definition for type-safe SQL queries with TypeScript errors when you reference a column or table that doesn't exist.

### Select 

The following example selects all rows of a `Comment` table. This returns the complete array of seeded development data from `db/seed.ts` which is then available for use in your page template:

```astro title="src/pages/index.astro"
---
import { db, Comment } from 'astro:db';

const comments = await db.select().from(Comment);
---

<h2>Comments</h2>

{
  comments.map(({ author, body }) => (
    <article>
      <p>Author: {author}</p>
      <p>{body}</p>
    </article>
  ))
}
```

<ReadMore>See the [Drizzle `select()` API reference](https://orm.drizzle.team/docs/select) for a complete overview.</ReadMore>

### Insert

To accept user input, such as handling form requests and inserting data into your remote hosted database, configure your Astro project for [on-demand rendering](/en/guides/on-demand-rendering/) and [add an adapter](/en/guides/on-demand-rendering/#add-an-adapter) for your deployment environment.

This example inserts a row into a `Comment` table based on a parsed form POST request:

```astro
---
// src/pages/index.astro
import { db, Comment } from 'astro:db';

if (Astro.request.method === 'POST') {
  // Parse form data
  const formData = await Astro.request.formData();
  const author = formData.get('author');
  const body = formData.get('body');
  if (typeof author === 'string' && typeof body === 'string') {
    // Insert form data into the Comment table
    await db.insert(Comment).values({ author, body });
  }
}

// Render the new list of comments on each request
const comments = await db.select().from(Comment);
---

<form method="POST" style="display: grid">
	<label for="author">Author</label>
	<input id="author" name="author" />

	<label for="body">Body</label>
	<textarea id="body" name="body"></textarea>

	<button type="submit">Submit</button>
</form>

<!-- Render `comments` -->
```

You can also use [Astro actions](/en/guides/actions/) to insert data into an Astro DB table. The following example inserts a row into a `Comment` table using an action:

```ts
// src/actions/index.ts
import { db, Comment } from 'astro:db';
import { defineAction } from 'astro:actions';
import { z } from 'astro:schema';

export const server = {
  addComment: defineAction({
    // Actions include type safety with Zod, removing the need
    // to check if typeof {value} === 'string' in your pages
    input: z.object({
      author: z.string(),
      body: z.string(),
    }),
    handler: async (input) => {
      const updatedComments = await db
        .insert(Comment)
        .values(input)
        .returning(); // Return the updated comments
      return updatedComments;
    },
  }),
};
```

<ReadMore>

See the [Drizzle `insert()` API reference](https://orm.drizzle.team/docs/insert) for a complete overview.

</ReadMore>

### Delete

You can also query your database from an API endpoint. This example deletes a row from a `Comment` table by the `id` parameter:

```ts
// src/pages/api/comments/[id].ts
import type { APIRoute } from "astro";
import { db, Comment, eq } from 'astro:db';

export const DELETE: APIRoute = async (ctx) => {
  await db.delete(Comment).where(eq(Comment.id, ctx.params.id ));
  return new Response(null, { status: 204 });
}
```

<ReadMore>

See the [Drizzle `delete()` API reference](https://orm.drizzle.team/docs/delete) for a complete overview.

</ReadMore>

### Filtering

To query for table results by a specific property, use [Drizzle options for partial selects](https://orm.drizzle.team/docs/select#partial-select). For example, add [a `.where()` call](https://orm.drizzle.team/docs/select#filtering) to your `select()` query and pass the comparison you want to make. 

The following example queries for all rows in a `Comment` table that contain the phrase "Astro DB." Use [the `like()` operator](https://orm.drizzle.team/docs/operators#like) to check if a phrase is present within the `body`:


```astro title="src/pages/index.astro"
---
import { db, Comment, like } from 'astro:db';

const comments = await db.select().from(Comment).where(
    like(Comment.body, '%Astro DB%')
);
---
```

### Drizzle utilities

All Drizzle utilities for building queries are exposed from the `astro:db` module. This includes:

- [Filter operators](https://orm.drizzle.team/docs/operators) like `eq()` and `gt()`
- [Aggregation helpers](https://orm.drizzle.team/docs/select#aggregations-helpers) like `count()`
- [The `sql` helper](https://orm.drizzle.team/docs/sql) for writing raw SQL queries

```ts
import { eq, gt, count, sql } from 'astro:db';
```

### Relationships

You can query related data from multiple tables using a SQL join. To create a join query, extend your `db.select()` statement with a join operator. Each function accepts a table to join with and a condition to match rows between the two tables.

This example uses an `innerJoin()` function to join `Comment` authors with their related `Author` information based on the `authorId` column. This returns an array of objects with each `Author` and `Comment` row as top-level properties:

```astro title="src/pages/index.astro"
---
import { db, eq, Comment, Author } from 'astro:db';

const comments = await db.select()
  .from(Comment)
  .innerJoin(Author, eq(Comment.authorId, Author.id));
---

<h2>Comments</h2>

{
  comments.map(({ Author, Comment }) => (
    <article>
      <p>Author: {Author.name}</p>
      <p>{Comment.body}</p>
    </article>
  ))
}
```

<ReadMore>

See the [Drizzle join reference](https://orm.drizzle.team/docs/joins#join-types) for all available join operators and config options.

</ReadMore>

### Batch Transactions

All remote database queries are made as a network request. You may need to "batch" queries together into a single transaction when making a large number of queries, or to have automatic rollbacks if any query fails.

This example seeds multiple rows in a single request using the `db.batch()` method:

```ts
// db/seed.ts
import { db, Author, Comment } from 'astro:db';

export default async function () {
  const queries = [];
  // Seed 100 sample comments into your remote database
  // with a single network request.
  for (let i = 0; i < 100; i++) {
    queries.push(db.insert(Comment).values({ body: `Test comment ${i}` }));
  }
  await db.batch(queries);
}
```

<ReadMore>

See the [Drizzle `db.batch()`](https://orm.drizzle.team/docs/batch-api) docs for more details.

</ReadMore>

## Pushing changes to your database

You can push changes made during development to your database.

### Pushing table schemas

Your table schema may change over time as your project grows. You can safely test configuration changes locally and push to your remote database when you deploy.

You can push your local schema changes to your remote database via the CLI using the `astro db push --remote` command:

<PackageManagerTabs>
  <Fragment slot="npm">
  ```sh
  npm run astro db push --remote
  ```
  </Fragment>
  <Fragment slot="pnpm">
  ```sh
  pnpm astro db push --remote
  ```
  </Fragment>
  <Fragment slot="yarn">
  ```sh
  yarn astro db push --remote
  ```
  </Fragment>
</PackageManagerTabs>

This command will verify that your local changes can be made without data loss and, if necessary, suggest how to safely make changes to your schema in order to resolve conflicts.

#### Pushing breaking schema changes

:::caution
__This will destroy your database__. Only perform this command if you do not need your production data.
:::

If you must change your table schema in a way that is incompatible with your existing data hosted on your remote database, you will need to reset your production database.

To push a table schema update that includes a breaking change, add the `--force-reset` flag to reset all production data:

<PackageManagerTabs>
  <Fragment slot="npm">
  ```sh
  npm run astro db push --remote --force-reset
  ```
  </Fragment>
  <Fragment slot="pnpm">
  ```sh
  pnpm astro db push --remote --force-reset
  ```
  </Fragment>
  <Fragment slot="yarn">
  ```sh
  yarn astro db push --remote --force-reset
  ```
  </Fragment>
</PackageManagerTabs>

### Renaming tables

It is possible to rename a table after pushing your schema to your remote database.

If you **do not have any important production data**, then you can [reset your database](#pushing-breaking-schema-changes) using the `--force-reset` flag. This flag will drop all of the tables in the database and create new ones so that it matches your current schema exactly.

To rename a table while preserving your production data, you must perform a series of non-breaking changes to push your local schema to your remote database safely.

The following example renames a table from `Comment` to `Feedback`:

<Steps>

1. In your database config file, add the `deprecated: true` property to the table you want to rename:

    ```ts title="db/config.ts" ins={2}
    const Comment = defineTable({
      deprecated: true,
    	columns: {
    		author: column.text(),
    		body: column.text(),
  		}
    });
    ```

2. Add a new table schema (matching the existing table's properties exactly) with the new name:

	  ```ts title="db/config.ts" ins={8-14}
    const Comment = defineTable({
        deprecated: true,
    	columns: {
    		author: column.text(),
    		body: column.text(),
  		}
    });
	  const Feedback = defineTable({
        columns: {
          author: column.text(),
          body: column.text(),
        }
    });
    ```

3. [Push to your remote database](#pushing-table-schemas) with `astro db push --remote`. This will add the new table and mark the old as deprecated.
4. Update any of your local project code to use the new table instead of the old table. You might need to migrate data to the new table as well.
5. Once you are confident that the old table is no longer used in your project, you can remove the schema from your `config.ts`:
		```ts title="db/config.ts" del={1-7}
    const Comment = defineTable({
          deprecated: true,
    	  columns: {
    		  author: column.text(),
    		  body: column.text(),
  		  }
    });

	  const Feedback = defineTable({
          columns: {
            author: column.text(),
            body: column.text(),
          }
    });
    ```
6. Push to your remote database again with `astro db push --remote`. The old table will be dropped, leaving only the new, renamed table.
</Steps>

### Pushing table data 

You may need to push data to your remote database for seeding or data migrations. You can author a `.ts` file with the `astro:db` module to write type-safe queries. Then, execute the file against your remote database using the command `astro db execute <file-path> --remote`:

The following Comments can be seeded using the command `astro db execute db/seed.ts --remote`:

```ts
// db/seed.ts
import { Comment } from 'astro:db';

export default async function () {
  await db.insert(Comment).values([
    { authorId: 1, body: 'Hope you like Astro DB!' },
    { authorId: 2, body: 'Enjoy!' },
  ])
}
```

<ReadMore>

See the [CLI reference](/en/guides/integrations-guide/db/#astro-db-cli-reference) for a complete list of commands.

</ReadMore>

## Building Astro DB integrations

[Astro integrations](/en/reference/integrations-reference/) can extend user projects with additional Astro DB tables and seed data.

Use the `extendDb()` method in the `astro:db:setup` hook to register additional Astro DB config and seed files.
The `defineDbIntegration()` helper provides TypeScript support and auto-complete for the `astro:db:setup` hook.

```js {8-13}
// my-integration/index.ts
import { defineDbIntegration } from '@astrojs/db/utils';

export default function MyIntegration() {
  return defineDbIntegration({
    name: 'my-astro-db-powered-integration',
    hooks: {
      'astro:db:setup': ({ extendDb }) => {
        extendDb({
          configEntrypoint: '@astronaut/my-package/config',
          seedEntrypoint: '@astronaut/my-package/seed',
        });
      },
      // Other integration hooks...
    },
  });
}
```

Integration [config](#define-your-database) and [seed](#seed-your-database-for-development) files follow the same format as their user-defined equivalents.

### Type safe operations in integrations

While working on integrations, you may not be able to benefit from Astro’s generated table types exported from `astro:db`.
For full type safety, use the `asDrizzleTable()` utility to create a table reference object you can use for database operations.

For example, given an integration setting up the following `Pets` database table:

```js
// my-integration/config.ts
import { defineDb, defineTable, column } from 'astro:db';

export const Pets = defineTable({
  columns: {
    name: column.text(),
    species: column.text(),
  },
});

export default defineDb({ tables: { Pets } });
```

The seed file can import `Pets` and use `asDrizzleTable()` to insert rows into your table with type checking:

```js {2,7} /typeSafePets(?! )/
// my-integration/seed.ts
import { asDrizzleTable } from '@astrojs/db/utils';
import { db } from 'astro:db';
import { Pets } from './config';

export default async function() {
  const typeSafePets = asDrizzleTable('Pets', Pets);

  await db.insert(typeSafePets).values([
    { name: 'Palomita', species: 'cat' },
    { name: 'Pan', species: 'dog' },
  ]);
}
```

The value returned by `asDrizzleTable('Pets', Pets)` is equivalent to `import { Pets } from 'astro:db'`, but is available even when Astro’s type generation can’t run.
You can use it in any integration code that needs to query or insert into the database.




## Migrate from Astro Studio to Turso

<Steps>

1. In the [Studio dashboard](https://studio.astro.build/), navigate to the project you wish to migrate. In the settings tab, use the "Export Database" button to download a dump of your database.
2. Follow the official instructions to [install the Turso CLI](https://docs.turso.tech/cli/installation) and [sign up or log in](https://docs.turso.tech/cli/authentication) to your Turso account.
3. Create a new database on Turso using the `turso db create` command.
   ```sh
   turso db create [database-name]
   ```
4. Fetch the database URL using the Turso CLI, and use it as the environment variable `ASTRO_DB_REMOTE_URL`.
   ```sh
   turso db show [database-name]
   ```
   ```dotenv
   ASTRO_DB_REMOTE_URL=[your-database-url]
   ```
5. Create a token to access your database, and use it as the environment variable `ASTRO_DB_APP_TOKEN`.
   ```sh
   turso db tokens create [database-name]
   ```
   ```dotenv
   ASTRO_DB_APP_TOKEN=[your-app-token]
   ```
6. Push your DB schema and metadata to the new Turso database.
   ```sh
   astro db push --remote
   ```
7. Import the database dump from step 1 into your new Turso DB.
   ```sh
   turso db shell [database-name] < ./path/to/dump.sql
   ```
8. Once you have confirmed your project connects to the new database, you can safely delete the project from Astro Studio.

</Steps>
