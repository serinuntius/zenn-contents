---
title: "ローカル開発環境"
---

# ローカル開発環境

Mastraエージェントを効果的に開発するには、適切なローカル開発環境を構築することが重要です。この章では、Mastraプロジェクトの作成方法、開発サーバーの起動と利用方法、外部サービスとの統合など、ローカル開発に関する詳細を解説します。

## プロジェクトの作成

Mastraプロジェクトを始めるには、主に2つの方法があります：

1. 新しいプロジェクトを作成する（`mastra create`）
2. 既存のプロジェクトにMastraを追加する（`mastra init`）

### 新しいプロジェクトの作成

新しいプロジェクトを作成するには、パッケージマネージャーまたはMastra CLIを使用できます：

```bash
# npmを使用する場合
npm create mastra@latest

# pnpmを使用する場合
pnpm create mastra

# yarnを使用する場合
yarn create mastra

# bunを使用する場合
bunx create-mastra
```

あるいは、Mastra CLIをグローバルにインストールして使用することもできます：

```bash
# npmの場合
npm install -g mastra@latest
mastra create

# pnpmの場合
pnpm add -g mastra@latest
mastra create
```

生成されるプロジェクト構造は以下のようになります：

```
my-project/
├── src/
│   └── mastra/
│       └── index.ts  # Mastraのエントリーポイント
├── package.json
└── tsconfig.json
```

### 既存のプロジェクトへの追加

既存のプロジェクトにMastraを追加するには、以下のコマンドを実行します：

```bash
mastra init
```

このコマンドは以下の変更を行います：

1. `src/mastra`ディレクトリとエントリーポイントを作成
2. 必要な依存関係を追加
3. TypeScriptコンパイラオプションを設定

### コマンド引数とインタラクティブセットアップ

プロジェクト作成時には、以下のような引数を指定できます：

```bash
--components    コンポーネントを指定（agents, memory, storage）
--llm-provider  LLMプロバイダーを指定（openai, anthropic）
--add-example   サンプル実装を含める
--llm-api-key   プロバイダーのAPIキー
```

引数なしでコマンドを実行すると、インタラクティブなCLIプロンプトが開始され、以下の項目を設定できます：

1. コンポーネントの選択
2. LLMプロバイダーの設定
3. APIキーの設定
4. サンプルコードの追加

### プロジェクトの初期化

プロジェクトを作成したら、以下のコマンドで依存関係をインストールし、開発サーバーを起動します：

```bash
# 依存関係のインストール
npm install

# 開発サーバーの起動（ポート3000）
mastra dev

# プレイグラウンドにアクセス: http://localhost:4111
```

## 開発サーバー（mastra dev）

`mastra dev`コマンドは、Mastraアプリケーションをローカルで提供する開発サーバーを起動します。このサーバーはエージェントとワークフローの検査と開発に必要な様々な機能を提供します。

### REST APIエンドポイント

`mastra dev`は、エージェントとワークフローのRESTful APIエンドポイントを提供します：

* `POST /api/agents/:agentId/generate` - エージェントに対してテキスト生成をリクエスト
* `POST /api/agents/:agentId/stream` - エージェントからのストリーミングレスポンスをリクエスト
* `POST /api/workflows/:workflowId/start` - ワークフローを開始
* `POST /api/workflows/:workflowId/:instanceId/event` - ワークフローインスタンスにイベントを送信
* `GET /api/workflows/:workflowId/:instanceId/status` - ワークフローインスタンスのステータスを取得

デフォルトでは、サーバーは`http://localhost:4111`で実行されますが、`--port`フラグを使用してポートを変更できます。

### クライアントSDKの使用

ローカルMastraサーバーと対話する最も簡単な方法は、TypeScript/JavaScriptクライアントSDKを使用することです：

```bash
npm install @mastra/client-js
```

その後、以下のようにローカルサーバーを指定して設定します：

```typescript
import { MastraClient } from "@mastra/client-js";

const client = new MastraClient({
  baseUrl: "http://localhost:4111",
});

// 例：ローカルエージェントとの対話
const agent = client.getAgent("my-agent");
const response = await agent.generate({
  messages: [{ role: "user", content: "こんにちは！" }],
});
```

クライアントSDKは、すべてのAPIエンドポイントに対して型安全なラッパーを提供し、ローカルのMastraアプリケーションの開発とテストを容易にします。

### UIプレイグラウンド

`mastra dev`は、以下の機能を持つUIを提供します：

* エージェントチャットインターフェース - エージェントとの対話テスト
* ワークフロー可視化ツール - ワークフローの状態と流れを視覚的に確認
* ツールプレイグラウンド - ツールの動作テスト

開発サーバーを起動すると、自動的にブラウザが開き、このUIにアクセスできます。

### OpenAPI仕様

開発サーバーはOpenAPI仕様も提供しており、以下のURLでアクセスできます：

* `GET /openapi.json`

この仕様を使用して、サードパーティのツールや独自のクライアントを構築することもできます。

## 外部サービスとの統合

Mastraは、様々な外部サービスとの統合機能を提供しています。これらの統合は、エージェントのツールやワークフローのステップとして使用できる、自動生成された型安全なAPIクライアントです。

### 統合のインストール

Mastraのデフォルト統合は、個別にインストール可能なnpmモジュールとしてパッケージ化されています。統合をプロジェクトに追加するには、npmを介してインストールし、Mastra設定にインポートします。

#### 例：GitHub統合の追加

1. **統合パッケージのインストール**

```bash
npm install @mastra/github
```

2. **プロジェクトへの統合の追加**

統合用の新しいファイル（例：`src/mastra/integrations/index.ts`）を作成し、統合をインポートします：

```typescript
// src/mastra/integrations/index.ts
import { GithubIntegration } from '@mastra/github';

export const github = new GithubIntegration({
  config: {
    PERSONAL_ACCESS_TOKEN: process.env.GITHUB_PAT!,
  },
});
```

`process.env.GITHUB_PAT!`を実際のGitHubパーソナルアクセストークンに置き換えるか、環境変数が適切に設定されていることを確認してください。

3. **ツールやワークフローでの統合の使用**

統合を使用してエージェントのツールを定義するか、ワークフローで使用できます：

```typescript
// src/mastra/tools/index.ts
import { createTool } from '@mastra/core';
import { z } from 'zod';
import { github } from '../integrations';

export const getMainBranchRef = createTool({
  id: 'getMainBranchRef',
  description: 'GitHubリポジトリからメインブランチの参照を取得',
  inputSchema: z.object({
    owner: z.string(),
    repo: z.string(),
  }),
  outputSchema: z.object({
    ref: z.string().optional(),
  }),
  execute: async ({ context }) => {
    const client = await github.getApiClient();
    const mainRef = await client.gitGetRef({
      path: {
        owner: context.owner,
        repo: context.repo,
        ref: 'heads/main',
      },
    });
    return { ref: mainRef.data?.ref };
  },
});
```

### エージェントでの統合の使用

作成したツールをエージェントに含めることで、エージェントが外部サービスと対話できるようになります：

```typescript
// src/mastra/agents/index.ts
import { openai } from '@ai-sdk/openai';
import { Agent } from '@mastra/core';
import { getMainBranchRef } from '../tools';

export const codeReviewAgent = new Agent({
  name: 'コードレビューエージェント',
  instructions: 'コードリポジトリをレビューしてフィードバックを提供するエージェント',
  model: openai('gpt-4o-mini'),
  tools: {
    getMainBranchRef,
    // 他のツール...
  },
});
```

### 環境設定

統合に必要なAPIキーやトークンが環境変数に適切に設定されていることを確認してください。例えば、GitHub統合の場合、GitHubパーソナルアクセストークンを設定する必要があります：

```
GITHUB_PAT=your_personal_access_token
```

機密情報を管理するには、`.env`ファイルなどの安全な方法を検討してください。

### 利用可能な統合

Mastraは、主にOAuthを必要としないAPIキーベースの統合を含む、いくつかのビルトイン統合を提供しています。利用可能な統合には、GitHub、Stripe、Resend、Firecrawlなどがあります。利用可能な統合の完全なリストについては、[Mastraのコードベース](https://github.com/mastra-ai/mastra/tree/main/integrations)または[npmパッケージ](https://www.npmjs.com/search?q=%22%40mastra%22)を確認してください。

## デバッグとテスト

ローカル開発環境では、エージェントやワークフローのデバッグとテストが容易に行えます。以下に、デバッグとテストのためのいくつかのヒントを紹介します。

### コンソールログの活用

ツールの実行時にコンソールログを追加して、実行の流れを追跡できます：

```typescript
export const myTool = createTool({
  // ... 他の設定 ...
  execute: async ({ context }) => {
    console.log('ツールが呼び出されました', context);
    // ツールのロジック
    const result = await doSomething(context.input);
    console.log('結果:', result);
    return { output: result };
  },
});
```

### エラーハンドリング

ツールやワークフローステップでエラー処理を追加して、問題を特定しやすくします：

```typescript
export const robustTool = createTool({
  // ... 他の設定 ...
  execute: async ({ context }) => {
    try {
      // ツールのロジック
      return { success: true, data: result };
    } catch (error) {
      console.error('ツールの実行中にエラーが発生しました:', error);
      return { success: false, error: error.message };
    }
  },
});
```

### UIプレイグラウンドでのテスト

開発サーバーのUIプレイグラウンドを使用して、エージェントとの対話をテストし、ツールの呼び出しと結果を確認します。これにより、エージェントが予期したように動作しているかを視覚的に確認できます。

## エージェント開発のベストプラクティス

ローカル開発環境でMastraエージェントを効果的に開発するためのベストプラクティスを以下に示します：

1. **モジュール化された構造を維持する**：
   - エージェント、ツール、統合、ワークフローを別々のファイルとディレクトリに整理します
   - 関連するコンポーネントをまとめてインポート/エクスポートします

2. **環境変数を使用する**：
   - 機密情報（APIキーなど）はコードにハードコーディングせず、環境変数を使用します
   - 開発と本番環境で異なる設定を使用できるように、環境変数を管理します

3. **型安全性を確保する**：
   - Zodスキーマを使用して、ツールの入力と出力を厳密に型付けします
   - TypeScriptの型チェックを最大限活用します

4. **段階的な開発とテスト**：
   - 複雑なエージェントを一度に構築するのではなく、ツールごとに段階的に開発してテストします
   - 各コンポーネントを独立してテストしてから、統合します

5. **ドキュメントを作成する**：
   - 各ツールとエージェントには明確な説明とドキュメントを提供します
   - 他の開発者が理解しやすいようにコードにコメントを追加します

## まとめ

この章では、Mastraのローカル開発環境について学びました。プロジェクトの作成から開発サーバーの使用、外部サービスとの統合まで、Mastraエージェントの開発に必要な環境を整える方法を詳細に解説しました。

適切なローカル開発環境を構築することで、エージェントの開発、テスト、デバッグを効率的に行うことができます。`mastra dev`コマンドが提供するツールとインターフェイスを活用して、本番環境にデプロイする前にエージェントの動作を確実に検証しましょう。

次の章では、Mastraアプリケーションを本番環境にデプロイする方法について詳しく見ていきます。 