---
title: "第10章：GitHub解析エージェントの完成"
---

# 🚀 ついに完成！GitHub解析エージェントで開発効率を爆上げしよう

## プロジェクトの全体像をサクッと確認

さあ、いよいよ完成の時がやってきました！ここまでの章で作り上げてきた様々なコンポーネントを組み合わせて、GitHub解析エージェントを完成させましょう。まずは、私たちが構築したプロジェクトの全体像を見てみましょう。これを見れば、あなたも「なるほど！こうつながっていたのか！」と感じるはずです。

```
src
└── mastra
    ├── agents
    │   └── index.ts
    ├── index.ts
    ├── models
    │   └── index.ts
    └── tools
        ├── analysis
        │   ├── readmeAnalyzer.ts
        │   ├── tokeiAnalyzer.ts
        │   └── treeAnalyzer.ts
        ├── cheatsheet
        │   ├── index.ts
        │   └── saveCheatsheet.ts
        ├── github
        │   └── cloneRepository.ts
        ├── index.ts
        └── rag
            ├── fileProcessor.ts
            └── vectorQuery.ts
```

驚くほどスッキリとした構造ですよね！各機能がきれいに整理されていて、一目でどのファイルが何をするのかがわかります。この明快な構造こそが、今後の拡張性を高め、メンテナンスを容易にする秘訣なんです。

## ツールのエクスポート方法（tools/index.ts）でスマートに連携

「バラバラのツールをどうやって一つのシステムとして動かすの？」と心配していましたか？大丈夫です！`tools/index.ts`ファイルが、その答えを教えてくれます。このファイルは、私たちが作った様々なツールをシンプルな形で提供してくれる、言わば「ツールの総合案内所」なんです。

```typescript
// GitHubツール
export { cloneRepositoryTool } from "./github/cloneRepository";

// 分析ツール
export { readmeAnalyzerTool } from "./analysis/readmeAnalyzer";
export { tokeiAnalyzerTool } from "./analysis/tokeiAnalyzer";
export { treeAnalyzerTool } from "./analysis/treeAnalyzer";

// RAGツール
export { fileProcessorTool } from "./rag/fileProcessor";

// ベクトルツール
export { vectorQueryTool } from "./rag/vectorQuery";

// チートシートツール
export { saveCheatsheetTool } from "./cheatsheet";
```

このシンプルな仕組みのおかげで、各ツールを簡単に組み合わせることができるんです。例えば、「GitHubからリポジトリを取得して→READMEを解析して→チートシートを作成する」といった複雑な処理も、まるでレゴブロックを組み立てるように簡単に実現できます。これぞMastraフレームワークの真髄と言えるでしょう！

## さあ、マスターエージェントを実際に動かしてみよう！✨

理論の説明はここまで！いよいよ実際にMastraが生み出すUIでエージェントを動かしてみましょう。Mastraの魅力は何と言っても「コードを書くだけで美しいUIが自動生成される」という驚きの機能です。つまり、フロントエンド開発の知識がなくても、プロフェッショナルなインターフェースが手に入るんです！

### 開発サーバーを立ち上げる（超カンタン！）

まずは開発サーバーを起動しましょう。ターミナルで以下のコマンドをタイプするだけです：

```bash
npm run dev
```

たったこれだけで、開発サーバーがポート4111で起動します。コンソールに表示される次のようなメッセージに、ちょっとワクワクしませんか？

```
Mastra app running at http://localhost:4111
```

### 美しいUIが現れる瞬間

ブラウザで `http://localhost:4111` にアクセスすると、まるで魔法のような体験が待っています。なんと、こんな素晴らしいUIが自動的に生成されているんです！

![Mastraフレームワークの自動生成UI](/images/4346a0fc6818f3/1-ui.png)

「これ、本当に自動生成されたの？」と疑ってしまうほどの完成度ですよね。このUIには次のような素晴らしい機能が詰まっています：

1. **エージェント選択機能** - 左サイドバーで「Cursor Rules生成エージェント」などの利用可能なエージェントを自由に選べます。複数のエージェントを切り替えるのも一瞬です！

2. **直感的なチャットインターフェース** - 中央部分では、まるで人間と会話するかのようにエージェントと対話できます。自然言語でリクエストを送るだけで、エージェントがリアルタイムで応答してくれるんです。

3. **便利なツールデバッグフォーム** - 各ツールには専用デバッグフォームがあり、パラメータを手動設定してテストできます。「このツールだけちょっと試したい」というときに超便利ですよ！

4. **ワークフローの可視化** - エージェントの処理の流れをリアルタイムで視覚的に表示。複雑な処理も一目で理解でき、問題点もすぐに発見できます。まるでプロジェクト管理ツールのような使い心地です！

5. **丸見えメモリと履歴** - 右サイドバーでは、エージェントが記憶している情報をリアルタイムで確認できます。エージェントの「頭の中」が丸見えなので、長時間の作業も安心して任せられますね。

6. **詳細なトレース機能** - エージェントがどのように考え、判断しているかを詳細に表示。AIのブラックボックスを透明化し、信頼感を高めます。「なぜこの判断をしたの？」という疑問が解消される瞬間です！

7. **プロンプト管理と最適化** - UIから直接システムプロンプトを編集可能で、AIが自動的に改善案を提案してくれます。技術者でなくても、エージェントの動作を自分好みに調整できるんです。これはすごい！

驚くべきことに、これらの機能すべてが自動生成されるんです！フロントエンド開発に時間を費やす必要はなく、あなたはツールとエージェントのロジックだけに集中できます。これこそが、Mastraフレームワークが提供する革命的な開発体験なんです！

### エージェントとの対話を体験しよう

さっそく、エージェントに話しかけてみましょう。例えば、以下のようなプロンプトを送ってみてください：

```
https://github.com/honojs/hono のリポジトリを分析して、Cursor用のルールセットを生成してください。
```

エージェントは瞬時にあなたの指示を理解し、次のような一連の処理を自動的に実行します：

1. GitHubリポジトリのクローン（バックグラウンドでサクッと完了）
2. READMEファイルの解析（プロジェクトの本質を把握）
3. 言語統計の収集（技術スタックを理解）
4. ディレクトリ構造の分析（設計思想を把握）
5. 重要ファイルの特定とベクトルDBへの格納（知識の構造化）
6. Cursor Rulesチートシートの生成（実用的なアウトプット）

そして驚くべきことに、これらの処理の進捗状況がリアルタイムで表示されるんです！まるでプロのアシスタントが隣で作業しているような安心感があります。しかも、最終的には便利なチートシートへのリンクまで提供してくれるんです。

これこそが、Mastraフレームワークの魅力です。複雑なツールの連携も、バックエンドの処理も、すべてがシームレスに統合され、ユーザーは自然な会話だけで高度な処理を実行できるのです。さあ、このエージェントを使って、あなたの開発効率も爆上げしましょう！


## 実際の対話内容

### Honoリポジトリの解析とチートシート作成の対話例

![Honoリポジトリの解析とチートシート作成の対話例](/images/4346a0fc6818f3/6-hono.png)

**ユーザー**:
> https://github.com/honojs/hono のチートシートを作成して！

**エージェント**:
```
Honoのチートシートを作成するために、まずリポジトリを分析していきます。段階的に処理を進めていきましょう。

リポジトリのクローンに成功しました。次にREADMEを解析して、Honoプロジェクトの目的と構造を理解していきます。

次に、Honoリポジトリの言語統計を分析して、どのような技術スタックで構成されているかを確認します。

続けて、Honoリポジトリのディレクトリ構造を分析していきます。

続けて、Honoリポジトリの解析を進めていきます。READMEとディレクトリ構造の分析から、Honoは小型で高速なWebフレームワークであり、さまざまなJavaScriptランタイム（Cloudflare Workers, Deno, Bun, Vercelなど）で動作することがわかりました。

次に、重要なファイルを特定し、そのコード内容を理解するため、まずはコアとなるファイル群をベクトルデータベースに保存していきます。
```

**ユーザー**:
> お願いします！
> 
> あくまで知りたいのはライブラリとしての外から見たインタフェースです。
> 内部実装までは必要ないのでそこは注意してください！
> 
> なるべく省エネで！

**エージェント**:
```
Honoフレームワークのチートシートを作成します。内部実装ではなく、API/インターフェースを中心に整理します。

すみません、もう少し具体的に進めていきます。Honoの主要なAPIと使い方を理解するために、重要なファイルを追加で調査します。

Honoの基本的な使い方を追加で調査します：
```

### Hono チートシート内容

#### 基本的な使い方
```ts
import { Hono } from 'hono'

// アプリケーションの作成
const app = new Hono()

// ルートハンドラの追加
app.get('/', (c) => c.text('Hello Hono!'))

export default app
```

#### ルーティング
```ts
// 基本的なルート
app.get('/users', (c) => c.json({ users: [] }))
app.post('/users', (c) => c.text('Created', 201))
app.put('/users/:id', (c) => c.text('Updated'))
app.delete('/users/:id', (c) => c.text('Deleted'))

// 複数のHTTPメソッドに対応
app.on(['GET', 'POST'], '/multi', (c) => c.text('Multi methods'))

// パラメータの取得
app.get('/users/:id', (c) => {
  const id = c.req.param('id')
  return c.text(`User ID: ${id}`)
})

// クエリパラメータ
app.get('/search', (c) => {
  const query = c.req.query('q')
  return c.text(`Search: ${query}`)
})

// グループ化
const api = new Hono()
api.get('/users', (c) => c.text('Users'))
api.get('/posts', (c) => c.text('Posts'))

app.route('/api', api) // すべてのルートに /api プレフィックスを追加
```

#### ミドルウェア
```ts
// グローバルミドルウェア
app.use('*', async (c, next) => {
  console.log('Request received')
  await next()
  console.log('Response sent')
})

// 特定のパスにミドルウェアを適用
app.use('/api/*', async (c, next) => {
  const token = c.req.header('Authorization')
  if (!token) return c.text('Unauthorized', 401)
  await next()
})

// 組み込みミドルウェア
import { logger, cors } from 'hono/middleware'

app.use(logger())
app.use('/api/*', cors())
```

#### リクエスト処理
```ts
// ヘッダーの取得
app.get('/headers', (c) => {
  const userAgent = c.req.header('User-Agent')
  return c.text(`User-Agent: ${userAgent}`)
})

// JSONボディの取得
app.post('/json', async (c) => {
  const body = await c.req.json()
  return c.json(body)
})

// フォームデータの取得
app.post('/form', async (c) => {
  const data = await c.req.formData()
  const name = data.get('name')
  return c.text(`Name: ${name}`)
})
```

#### レスポンス生成
```ts
// テキストレスポンス
app.get('/text', (c) => c.text('Hello'))

// JSONレスポンス
app.get('/json', (c) => c.json({ message: 'Hello' }))

// HTMLレスポンス
app.get('/html', (c) => c.html('<h1>Hello</h1>'))

// カスタムレスポンス
app.get('/custom', (c) => {
  return new Response('Custom response', {
    status: 200,
    headers: {
      'Content-Type': 'text/plain',
      'X-Custom': 'Value'
    }
  })
})

// リダイレクト
app.get('/redirect', (c) => c.redirect('/new-location'))
```

#### JSX サポート
```tsx
import { Hono } from 'hono'
import { jsx } from 'hono/jsx'

const app = new Hono()

// JSXレンダリング
app.get('/', (c) => {
  return c.html(
    <html>
      <body>
        <h1>Hello Hono!</h1>
      </body>
    </html>
  )
})

// コンポーネント
const Layout = (props: { children: any }) => {
  return (
    <html>
      <body>{props.children}</body>
    </html>
  )
}

app.get('/components', (c) => {
  return c.html(
    <Layout>
      <h1>Hono with Components</h1>
    </Layout>
  )
})
```

#### エラーハンドリング
```ts
// エラーハンドラ
app.onError((err, c) => {
  console.error(err)
  return c.text('Internal Server Error', 500)
})

// 404ハンドラ
app.notFound((c) => {
  return c.text('Not Found', 404)
})
```

#### 環境変数と型定義
```ts
// 環境変数の型を定義
type Bindings = {
  DATABASE_URL: string
  API_KEY: string
}

// 型付きHonoアプリケーション
const app = new Hono<{ Bindings: Bindings }>()

app.get('/db', (c) => {
  const dbUrl = c.env.DATABASE_URL
  return c.text(`DB URL: ${dbUrl}`)
})
```

#### アダプター
```ts
// Cloudflare Workers
export default app

// Bun
import { serve } from '@hono/node-server'
serve(app)

// Node.js
import { serve } from '@hono/node-server'
serve(app)

// Deno
import { serve } from '@hono/deno-server'
serve(app)

// Vercel
import { handle } from '@hono/vercel'
export default handle(app)

// AWS Lambda
import { handle } from '@hono/aws-lambda'
export const handler = handle(app)
```

**ユーザー**:
> ではCursorルールとしてAIにコーディングさせるときに気をつける点をしっかりと考えて、./output/hono-new.mdに保存してください。

**エージェント**:
```
Honoフレームワークを使用する際のCursorルールを作成し、保存します。

Honoフレームワークのためのカーソルルールを作成し、./output/hono-rules.md に保存しました。このチートシートには以下の内容を含めました：

基本的なインポートとアプリケーション構築のルール
適切なコード構造とルーティングの書き方
正しいレスポンス生成方法
非同期処理のベストプラクティス
パラメータ処理の最適な方法
JSX使用時の注意点
各ランタイム(Cloudflare Workers, Bun, Node.js, Deno)固有の考慮事項
エラーハンドリングとセキュリティのガイドライン
デバッグとテストの手法

追加で必要な情報やルールがあれば、お知らせください。
```

## 実際にファイル出力されたチートシート

# Hono Cursor Rules チートシート

## 基本ルール

1. **最新のインポート方法を使用する**
   - JSR形式を優先: `@hono/hono` (deno.land/xは非推奨)
   - npm: `hono` パッケージを使用

2. **正しい型付けを常に行う**
   - 環境変数には明示的な型定義を提供
   - ハンドラのパラメータおよび戻り値の型を明示

3. **適切なエクスポート形式を使用する**
   - デフォルトエクスポートを使用: `export default app`
   - アダプタを使用する場合は適切にラップする

## コード構造

1. **アプリケーション初期化**
```ts
import { Hono } from 'hono'

// 環境変数の型付け例
type Bindings = {
  DATABASE_URL: string
  API_KEY: string
}

// 型付きアプリケーション
const app = new Hono<{ Bindings: Bindings }>()

// ルート定義
app.get('/', (c) => c.text('Hello Hono!'))

export default app
```

2. **ルーティングはメソッドチェーンで整理する**
```ts
// 良い例
app
  .get('/', (c) => c.text('Home'))
  .get('/about', (c) => c.text('About'))
  .post('/api/users', (c) => c.text('Create user'))

// サブルーティングはグループ化する
const api = new Hono()
api
  .get('/users', (c) => c.json([{ id: 1 }]))
  .post('/users', (c) => c.text('Created', 201))

app.route('/api', api)
```

3. **ミドルウェアは的確に配置する**
```ts
// グローバルミドルウェア
app.use('*', logger())

// 特定のパス・メソッドに対するミドルウェア
app.use('/api/*', auth())

// 上から下へ順番に実行される点に注意
```

## レスポンス生成

1. **コンテキストメソッドを使用してレスポンスを返す**
```ts
// 推奨
app.get('/api/users', (c) => c.json({ users: [] }))

// 非推奨 (明示的なコンテキストメソッドを使わない)
app.get('/api/users', () => {
  return new Response(JSON.stringify({ users: [] }), {
    headers: { 'Content-Type': 'application/json' }
  })
})
```

2. **エラーレスポンスは統一フォーマットを使用**
```ts
app.onError((err, c) => {
  console.error(err)
  return c.json({ 
    error: {
      message: err.message,
      status: err instanceof HTTPException ? err.status : 500
    }
  }, err instanceof HTTPException ? err.status : 500)
})
```

## 非同期処理

1. **非同期処理は適切に await する**
```ts
// 良い例
app.post('/api/users', async (c) => {
  const data = await c.req.json()
  // データベース処理など
  return c.json({ id: 123, ...data }, 201)
})

// 非推奨 (Promiseのハンドリングが不適切)
app.post('/api/users', (c) => {
  c.req.json().then(data => {
    // 非同期処理の結果がレスポンスに反映されない
  })
  return c.text('OK')
})
```

## パラメータ処理

1. **URLパラメータとクエリパラメータ**
```ts
// 良い例
app.get('/users/:id', (c) => {
  const id = c.req.param('id')
  const limit = c.req.query('limit') || '10'
  // ...
})

// 型安全なパラメータ取得 (validator使用)
app.get('/users/:id', validator('param', { id: 'number' }), (c) => {
  const id = c.req.valid('param').id // 数値型として取得可能
  // ...
})
```

## JSXの使用

1. **JSXを使用する場合は正しいインポートを行う**
```ts
import { Hono } from 'hono'
import { jsx } from 'hono/jsx'
// または
/** @jsx jsx */
import { jsx } from 'hono/jsx'
```

2. **コンポーネントは適切に分割する**
```tsx
// ルート定義とコンポーネント定義を分離する
const Layout = (props: { children: any }) => (
  <html>
    <head><title>Hono App</title></head>
    <body>{props.children}</body>
  </html>
)

app.get('/', (c) => {
  return c.html(<Layout><h1>Hello Hono!</h1></Layout>)
})
```

## ランタイム固有の注意点

1. **Cloudflare Workers**
```ts
// 環境変数の型定義
type Bindings = {
  KV: KVNamespace
  SECRET: string
}

const app = new Hono<{ Bindings: Bindings }>()

app.get('/kv', async (c) => {
  const value = await c.env.KV.get('key')
  return c.text(value || 'Not found')
})

export default app
```

2. **Bun / Node.js**
```ts
import { serve } from '@hono/node-server'
// または Bunの場合
import { serve } from '@hono/bun-server'

const app = new Hono()
// ...

// 開発環境の場合
if (import.meta.env?.DEV) {
  serve(app)
}

export default app
```

3. **Deno**
```ts
import { serve } from '@hono/deno-server'

const app = new Hono()
// ...

serve(app)
```

## ベストプラクティス

1. **ビジネスロジックとハンドラを分離する**
```ts
// service層を作成
const userService = {
  getUsers: async () => {
    // ビジネスロジック
    return [{ id: 1, name: 'User 1' }]
  }
}

// ハンドラで使用
app.get('/api/users', async (c) => {
  const users = await userService.getUsers()
  return c.json(users)
})
```

2. **エラーハンドリングを常に行う**
```ts
app.get('/api/users/:id', async (c) => {
  const id = c.req.param('id')
  
  try {
    const user = await userService.getUserById(id)
    if (!user) {
      return c.json({ error: 'User not found' }, 404)
    }
    return c.json(user)
  } catch (error) {
    // エラーログ記録
    console.error(`Error fetching user ${id}:`, error)
    return c.json({ error: 'Failed to fetch user' }, 500)
  }
})
```

3. **適切なHTTPステータスコードを使用する**
```ts
// リソース作成 - 201 Created
app.post('/api/users', async (c) => {
  const user = await createUser(await c.req.json())
  return c.json(user, 201)
})

// リソース更新 - 200 OK または 204 No Content
app.put('/api/users/:id', async (c) => {
  // 204を返す場合
  await updateUser(c.req.param('id'), await c.req.json())
  return c.text('', 204)
})

// リソース部分更新 - 200 OK
app.patch('/api/users/:id', async (c) => {
  const user = await partialUpdateUser(c.req.param('id'), await c.req.json())
  return c.json(user)
})

// リソース削除 - 204 No Content
app.delete('/api/users/:id', async (c) => {
  await deleteUser(c.req.param('id'))
  return c.text('', 204)
})
```

4. **セキュリティへの配慮**
```ts
import { csrf, secureHeaders } from 'hono/middleware'

// CSRFトークン
app.use('/form-actions/*', csrf())

// セキュリティヘッダの適用
app.use('*', secureHeaders())

// CORS設定
app.use('/api/*', cors({
  origin: ['https://example.com'],
  allowMethods: ['GET', 'POST', 'PUT', 'DELETE'],
  maxAge: 600,
}))
```

5. **パフォーマンスの最適化**
```ts
import { cache } from 'hono/cache'

// キャッシュの適用
app.get('/api/products', cache({
  cacheName: 'products-cache',
  cacheControl: 'max-age=60',
}), async (c) => {
  // 重い処理
  const products = await fetchAllProducts()
  return c.json(products)
})
```

## デバッグとテスト

1. **ロギングを適切に行う**
```ts
import { logger } from 'hono/logger'

// 開発環境のみloggerを有効化
if (process.env.NODE_ENV === 'development') {
  app.use('*', logger())
}

app.get('/api/resource', async (c) => {
  console.log('Fetching resource')
  // ...
})
```

2. **テスト用のユーティリティを活用**
```ts
import { describe, it, expect } from 'vitest'
import { Hono } from 'hono'

describe('API Routes', () => {
  it('should return users', async () => {
    const app = new Hono()
    app.get('/api/users', (c) => c.json([{ id: 1 }]))
    
    const res = await app.request('/api/users')
    expect(res.status).toBe(200)
    
    const data = await res.json()
    expect(data).toHaveLength(1)
  })
})
```

このように、ユーザーからのシンプルな指示から始まり、エージェントがGitHubリポジトリを分析し、Honoフレームワークの使い方を整理したチートシートを作成し、最終的にはAI支援コーディングのためのCursorルールとして保存するまでの一連の対話プロセスを見ることができます。