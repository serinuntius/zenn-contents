---
title: "デプロイメント"
---

# デプロイメント

Mastraで開発したAIエージェントやワークフローをテスト環境から本番環境へと移行し、実際のユーザーが利用できるようにするためのデプロイメント方法について解説します。この章では、Mastraアプリケーションのデプロイ方法、サーバー設定、ロギングとトレーシングの設定など、本番環境への移行に必要な知識を学びます。

## Mastraアプリケーションのデプロイ概要

Mastraアプリケーションは主に2つの方法でデプロイできます：

1. **プラットフォーム直接デプロイ**: Cloudflare Workers、Vercel、Netlifyなどの特定プラットフォーム向けのデプロイヤーを使用
2. **ユニバーサルデプロイ**: `mastra build`コマンドを使用して標準的なNode.jsサーバーを生成し、あらゆる環境で実行

### デプロイの前提条件

デプロイを始める前に、以下の準備が必要です：

* Node.js（バージョン18以上推奨）がインストールされていること
* プラットフォーム固有のデプロイヤーを使用する場合：
  * 選択したプラットフォームのアカウント
  * 必要なAPIキーや認証情報

## プラットフォーム直接デプロイ

Mastraは、以下のプラットフォーム向けの特定デプロイヤーを提供しています：

* **Cloudflare Workers**
* **Vercel**
* **Netlify**

これらのデプロイヤーは、構成とデプロイプロセスを自動化し、プラットフォーム固有の機能を最大限に活用できるようにします。

### デプロイヤーのインストール

使用したいプラットフォームに応じてデプロイヤーをインストールします：

```bash
# Cloudflare用
npm install @mastra/deployer-cloudflare

# Vercel用
npm install @mastra/deployer-vercel

# Netlify用
npm install @mastra/deployer-netlify
```

### デプロイヤーの設定

エントリーファイル（通常は`src/mastra/index.ts`）でデプロイヤーを設定します：

```typescript
import { Mastra, createLogger } from '@mastra/core';
import { CloudflareDeployer } from '@mastra/deployer-cloudflare';

export const mastra = new Mastra({
  agents: { /* エージェント設定 */ },
  logger: createLogger({
    name: 'MyApp',
    level: 'debug'
  }),
  deployer: new CloudflareDeployer({
    scope: 'your-cloudflare-account-id',
    projectName: 'your-project-name',
    // 詳細な設定オプションはリファレンスドキュメントを参照
  }),
});
```

### プラットフォーム別デプロイヤー設定

各デプロイヤーには固有の設定オプションがあります。以下に基本的な例を示します：

#### Cloudflareデプロイヤー

```typescript
new CloudflareDeployer({
  scope: 'your-cloudflare-account-id',
  projectName: 'your-project-name',
});
```

#### Vercelデプロイヤー

```typescript
new VercelDeployer({
  teamId: 'your-vercel-team-id',
  projectName: 'your-project-name',
  token: 'your-vercel-token'
});
```

#### Netlifyデプロイヤー

```typescript
new NetlifyDeployer({
  scope: 'your-netlify-team-slug',
  projectName: 'your-project-name',
  token: 'your-netlify-token'
});
```

## ユニバーサルデプロイ

MastraはスタンダードなNode.jsサーバーにビルドされるため、Node.jsアプリケーションを実行できるあらゆるプラットフォームにデプロイできます：

* クラウドVM（AWS EC2、DigitalOcean Droplets、GCP Compute Engine）
* コンテナプラットフォーム（Docker、Kubernetes）
* Platform as a Service（Heroku、Railway）
* 自己ホスト型サーバー

### ビルドプロセス

アプリケーションをビルドするには、以下のコマンドを実行します：

```bash
# カレントディレクトリからビルド
mastra build

# または特定のディレクトリを指定
mastra build --dir ./my-project
```

ビルドプロセスは以下の手順で進行します：

1. エントリーファイル（`src/mastra/index.ts`または`src/mastra/index.js`）の特定
2. `.mastra`出力ディレクトリの作成
3. Rollupを使用したコードのバンドル化（ツリーシェイキングとソースマップ付き）
4. [Hono](https://hono.dev/)ベースのHTTPサーバーの生成

### サーバーの実行

ビルドされたHTTPサーバーを起動するには：

```bash
node .mastra/index.js
```

## Mastraサーバーの設定

Mastraアプリケーションをデプロイすると、エージェント、ワークフロー、その他の機能をAPIエンドポイントとして公開するHTTPサーバーとして実行されます。

### サーバーアーキテクチャ

Mastraは、基盤となるHTTPサーバーフレームワークとして[Hono](https://hono.dev/)を使用しています。`mastra build`コマンドでビルドすると、`.mastra`ディレクトリにHonoベースのHTTPサーバーが生成されます。

このサーバーは以下を提供します：

* 登録されたすべてのエージェント用のAPIエンドポイント
* 登録されたすべてのワークフロー用のAPIエンドポイント
* カスタムミドルウェアのサポート

### サーバーミドルウェア

Mastraでは、APIルートに適用するカスタムミドルウェア関数を設定できます。これは認証、ロギング、CORSなどのHTTPレベルの機能をAPIエンドポイントに追加するのに便利です。

```typescript
import { Mastra } from '@mastra/core';

export const mastra = new Mastra({
  // その他の設定オプション
  serverMiddleware: [
    {
      handler: async (c, next) => {
        // 例：認証チェックを追加
        const authHeader = c.req.header('Authorization');
        if (!authHeader) {
          return new Response('Unauthorized', { status: 401 });
        }
        // 次のミドルウェアまたはルートハンドラに進む
        await next();
      },
      path: '/api/*', // オプション：指定しない場合、デフォルトは '/api/*'
    },
    {
      handler: async (c, next) => {
        // 例：リクエストロギングを追加
        console.log(`${c.req.method} ${c.req.url}`);
        await next();
      },
      // パスを指定しない場合、このミドルウェアはすべてのルートに適用される
    }
  ]
});
```

#### 一般的なミドルウェアの使用例

##### 認証

```typescript
{
  handler: async (c, next) => {
    const authHeader = c.req.header('Authorization');
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return new Response('Unauthorized', { status: 401 });
    }
    const token = authHeader.split(' ')[1];
    // ここでトークンを検証
    await next();
  },
  path: '/api/*',
}
```

##### CORSサポート

```typescript
{
  handler: async (c, next) => {
    // CORSヘッダーを追加
    c.header('Access-Control-Allow-Origin', '*');
    c.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
    c.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    
    // プリフライトリクエストを処理
    if (c.req.method === 'OPTIONS') {
      return new Response(null, { status: 204 });
    }
    await next();
  }
}
```

##### リクエストロギング

```typescript
{
  handler: async (c, next) => {
    const start = Date.now();
    await next();
    const duration = Date.now() - start;
    console.log(`${c.req.method} ${c.req.url} - ${duration}ms`);
  }
}
```

## ロギングとトレーシング

効果的なロギングとトレーシングは、アプリケーションの動作を理解するために不可欠です。特にAIエンジニアリングでは、トレーシングが重要です。Mastraのテレメトリ機能を使用すると、すべての実行のすべてのステップの入出力を可視化できます。

### ロギング

Mastraのログは、特定の関数がいつ実行されるか、どのような入力データを受け取るか、どのように応答するかなどの詳細を記録します。

#### 基本設定

以下は、`INFO`レベルでコンソールロガーを設定する最小限の例です。これにより、情報メッセージ以上（つまり、`INFO`、`WARN`、`ERROR`）がコンソールに出力されます：

```typescript
import { Mastra } from "@mastra/core";
import { createLogger } from "@mastra/core/logger";

export const mastra = new Mastra({
  // その他のMastra設定...
  logger: createLogger({
    name: "Mastra",
    level: "info",
  }),
});
```

この設定では：
* `name: "Mastra"`はログをグループ化する名前を指定します。
* `level: "info"`は記録するログの最小重要度を設定します。

### テレメトリ

Mastraは、アプリケーションのトレースとモニタリングにOpenTelemetry Protocol（OTLP）をサポートしています。テレメトリが有効になると、Mastraはエージェント操作、LLM対話、ツール実行、統合呼び出し、ワークフロー実行、データベース操作などのすべてのコア機能を自動的にトレースします。

#### 基本設定

テレメトリを有効にする簡単な例：

```typescript
export const mastra = new Mastra({
  // ... その他の設定
  telemetry: {
    serviceName: "my-app",
    enabled: true,
    sampling: {
      type: "always_on",
    },
    export: {
      type: "otlp",
      endpoint: "http://localhost:4318", // SigNoz ローカルエンドポイント
    },
  },
});
```

#### 環境変数による設定

環境変数を通じてOTLPエンドポイントとヘッダーを設定することもできます：

```
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
OTEL_EXPORTER_OTLP_HEADERS=x-api-key=your-api-key
```

そして設定ファイルでは：

```typescript
export const mastra = new Mastra({
  // ... その他の設定
  telemetry: {
    serviceName: "my-app",
    enabled: true,
    export: {
      type: "otlp",
      // エンドポイントとヘッダーは環境変数から取得される
    },
  },
});
```

## 環境変数

デプロイ時には、以下のような環境変数の設定が必要です：

1. プラットフォームデプロイヤー変数（プラットフォームデプロイヤーを使用する場合）：
   * プラットフォームの認証情報

2. エージェントAPIキー：
   * `OPENAI_API_KEY`
   * `ANTHROPIC_API_KEY`

3. サーバー設定（ユニバーサルデプロイ用）：
   * `PORT`: HTTPサーバーポート（デフォルト: 3000）
   * `HOST`: サーバーホスト（デフォルト: 0.0.0.0）

## デプロイのベストプラクティス

Mastraアプリケーションを本番環境に効果的にデプロイするためのベストプラクティスをいくつか紹介します：

### 1. セキュリティ対策

* 認証ミドルウェアを実装して、APIエンドポイントへの不正アクセスを防止
* 環境変数を使用してAPIキーや機密情報を管理
* HTTPS接続を強制
* APIレート制限を実装して、過負荷やDoS攻撃から保護

### 2. パフォーマンス最適化

* プロダクション環境ではロギングレベルを`info`または`warn`に設定
* 必要なトレースだけをサンプリングして、テレメトリのオーバーヘッドを最小化
* 長時間実行されるワークフローには非同期処理を使用

### 3. 監視とアラート

* テレメトリデータを監視システムに統合
* 重要なエラーやパフォーマンス低下に関するアラートを設定
* リソース使用量（メモリ、CPU、ネットワーク）を定期的に監視

### 4. デプロイプロセス

* CI/CDパイプラインを設定して自動デプロイを実現
* デプロイ前にテストを実行して、問題を早期に発見
* ブルー/グリーンデプロイメントまたはカナリアリリースを検討して、リスクを軽減

### 5. スケーリング

* 需要に応じてアプリケーションを水平スケーリングできるアーキテクチャを採用
* ステートレスな設計を維持し、分散システムとの互換性を確保
* 外部サービス（データベースなど）のスケーリング計画も検討

## まとめ

この章では、Mastraアプリケーションを本番環境にデプロイするための様々な方法と設定について学びました。プラットフォーム固有のデプロイヤーを使用する方法とユニバーサルデプロイ方法、サーバー設定のカスタマイズ、効果的なロギングとトレーシングの設定など、本番環境でのMastraアプリケーションの運用に必要な知識を解説しました。

適切なデプロイ戦略と設定を選択することで、Mastraで開発したAIエージェントとワークフローが安定して高性能に動作し、ユーザーに価値を提供できるようになります。次の章では、Mastraアプリケーションの評価（Evals）について学び、AIエージェントのパフォーマンスを測定・改善する方法を探ります。 