---
title: "ローカル開発環境"
---

# ローカル開発環境でAIエージェントを構築しよう 🛠️

## ローカル開発の重要性 🏠

AIエージェント開発において、ローカル開発環境は非常に重要です。本番環境にデプロイする前に、自分のコンピュータ上でエージェントの動作を確認し、迅速に改良を重ねることができます。Mastraは、このローカル開発プロセスを簡単かつ効率的にするための優れたツールを提供しています。

ローカル開発環境を使うことで得られる主なメリットは：

- **迅速な開発サイクル** ⚡: 変更をすぐに確認でき、開発スピードが格段に向上します
- **コスト削減** 💰: API呼び出しの回数を減らせるため、開発コストを抑えられます
- **プライバシー保護** 🔒: センシティブなデータを外部に送信せずに開発できます
- **ネットワーク非依存** 🌐: インターネット接続がなくても開発を続けられます
- **実験の自由度** 🧪: 大胆な実験も気軽に試せます

それでは、Mastraのローカル開発環境を構築し、実際にエージェントを動かしてみましょう！

## Mastra CLIのインストール 🔧

Mastraのローカル開発環境を構築するには、まずMastra CLIをインストールする必要があります。これは、エージェントの開発、テスト、デプロイを支援する強力なコマンドラインツールです。

```bash
# npmを使ってグローバルにインストール
npm install -g @mastra/cli

# インストールの確認
mastra --version
```

インストールが完了したら、`mastra`コマンドが使えるようになります。このツールを使って、プロジェクトの初期化からデプロイまで、様々な操作を行うことができます。

## プロジェクトの作成 🏗️

新しいMastraプロジェクトを作成するには、`mastra init`コマンドを使います。このコマンドは、必要なファイルとフォルダ構造を自動的に生成します。

```bash
# 新しいディレクトリを作成してプロジェクトを初期化
mkdir my-agent-project
cd my-agent-project
mastra init

# または直接プロジェクト名を指定して初期化
mastra init my-agent-project
```

プロジェクト初期化時には、いくつかの質問に答える必要があります：

- プロジェクト名
- 使用するAIプロバイダー（OpenAI、Anthropic、Cohereなど）
- プロジェクトの説明
- 使用するテンプレート（基本エージェント、RAGエージェント、ワークフローなど）

質問に答えると、Mastraは自動的にプロジェクトの基本構造を生成します。

## プロジェクト構造の理解 📂

生成されたプロジェクトの基本構造は以下のようになっています：

```
my-agent-project/
├── .env                  # 環境変数（APIキーなど）
├── .gitignore            # Gitの除外ファイル設定
├── package.json          # プロジェクト設定とパッケージ依存関係
├── tsconfig.json         # TypeScript設定
├── src/                  # ソースコードディレクトリ
│   ├── index.ts          # メインエントリーポイント
│   ├── agent.ts          # エージェント定義
│   └── tools/            # カスタムツール
├── tests/                # テストファイル
└── README.md             # プロジェクト説明
```

この構造は、選択したテンプレートによって若干異なる場合があります。例えば、RAGエージェントを選択した場合は、ベクトルストアの設定ファイルなども含まれます。

## 環境変数の設定 🔐

エージェントを動かすには、必要なAPIキーを設定する必要があります。プロジェクトルートにある`.env`ファイルを編集して、必要なAPIキーを追加しましょう。

```
# .envファイルの例
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-...
PINECONE_API_KEY=...
PINECONE_ENVIRONMENT=...
```

これらのAPIキーは、各サービスのウェブサイトから取得できます。APIキーは秘密情報なので、Gitなどのバージョン管理システムにコミットしないように注意しましょう（`.gitignore`ファイルに`.env`が含まれていることを確認してください）。

## エージェントの定義 🤖

次に、`src/agent.ts`ファイルを編集して、エージェントの動作を定義します。基本的なエージェント定義は以下のようになります：

```typescript
import { Agent } from "@mastra/core/agent";
import { openai } from "@ai-sdk/openai";

export const myAgent = new Agent({
  name: "ヘルパーエージェント",
  instructions: `あなたは親切で役立つアシスタントです。
ユーザーの質問に簡潔かつ正確に回答してください。
わからないことがあれば、正直に「わかりません」と答えてください。`,
  model: openai("gpt-4o"),
});
```

この例では、OpenAIのGPT-4oモデルを使用したシンプルなエージェントを定義しています。`instructions`フィールドには、エージェントの役割や動作方針を詳細に記述します。これはエージェントのパーソナリティと能力を決定する重要な部分です。

## カスタムツールの追加 🧰

エージェントの能力を拡張するには、カスタムツールを追加します。ツールを使うことで、エージェントは外部APIの呼び出し、データベースの検索、計算の実行など、様々なタスクを実行できるようになります。

```typescript
import { Agent } from "@mastra/core/agent";
import { openai } from "@ai-sdk/openai";
import { z } from "zod";

// 天気情報を取得するカスタムツール
const getWeatherTool = {
  id: "getWeather",
  description: "特定の都市の現在の天気情報を取得します",
  schema: z.object({
    city: z.string().describe("天気を調べたい都市名"),
    country: z.string().optional().describe("国名（省略可）"),
  }),
  execute: async ({ city, country }) => {
    // 実際には外部APIを呼び出すコードを書きます
    console.log(`${city}, ${country || "Japan"}の天気を取得中...`);
    
    // デモ用のダミーレスポンス
    return {
      temperature: 22,
      condition: "晴れ",
      humidity: 65,
      windSpeed: 10,
    };
  },
};

// ツールを使用するエージェントの定義
export const weatherAgent = new Agent({
  name: "天気アシスタント",
  instructions: `あなたは天気情報を提供するアシスタントです。
ユーザーが天気について質問したら、getWeatherツールを使って情報を取得し、
わかりやすく回答してください。`,
  model: openai("gpt-4o"),
  tools: {
    getWeatherTool,
  },
});
```

この例では、特定の都市の天気情報を取得するカスタムツールを定義しています。実際のアプリケーションでは、このツールは外部の天気APIを呼び出すコードを含むことになります。

## ローカルサーバーの起動 🚀

エージェントの定義が完了したら、ローカル開発サーバーを起動して動作を確認しましょう。Mastra CLIの`dev`コマンドを使うと、ホットリロード機能付きの開発サーバーが起動します。

```bash
# 開発サーバーを起動
mastra dev
```

このコマンドを実行すると、以下のような出力が表示されます：

```
🚀 Mastra開発サーバーを起動中...
📡 サーバーが http://localhost:3000 で起動しました
🔌 WebSocketサーバーが ws://localhost:3001 で起動しました
👀 ファイルの変更を監視中...
```

これで、ブラウザから`http://localhost:3000`にアクセスすると、エージェントとチャットできるインターフェースが表示されます。

## デバッグとテスト 🐛

開発中は、エージェントの動作を詳細に確認するためのデバッグ機能が重要です。Mastraは、ログ出力やステップ実行などの便利なデバッグ機能を提供しています。

### ログの確認

開発サーバーのコンソール出力には、エージェントの動作に関する詳細なログが表示されます。これには、モデルへのリクエスト、ツールの実行結果、エラーメッセージなどが含まれます。

### デバッグモードの有効化

より詳細なデバッグ情報を表示するには、環境変数`DEBUG=mastra:*`を設定します：

```bash
# デバッグモードを有効にして開発サーバーを起動
DEBUG=mastra:* mastra dev
```

これにより、内部処理の詳細なログが表示されるようになります。

### 自動テストの作成

エージェントの動作を自動的にテストするには、`tests`ディレクトリにテストファイルを作成します。Mastraは、Jest、Vitest、Mochaなどの一般的なJavaScriptテストフレームワークと互換性があります。

```typescript
// tests/agent.test.ts
import { describe, it, expect } from 'vitest';
import { myAgent } from '../src/agent';

describe('MyAgent', () => {
  it('should respond to a greeting', async () => {
    const response = await myAgent.generate('こんにちは！');
    expect(response.text).toContain('こんにちは');
  });

  it('should use the weather tool when asked about weather', async () => {
    const response = await myAgent.generate('東京の天気を教えてください');
    expect(response.toolCalls).toHaveLength(1);
    expect(response.toolCalls[0].id).toBe('getWeather');
  });
});
```

テストを実行するには、以下のコマンドを使います：

```bash
# テストを実行
npm test
```

## 高度な機能：ローカルモデルの使用 🧠

Mastraは、OpenAIやAnthropicなどのクラウドベースのAIモデルだけでなく、ローカルで実行できるAIモデルもサポートしています。これにより、インターネット接続なしで開発したり、センシティブなデータを外部に送信せずに処理したりすることができます。

### Ollama統合

[Ollama](https://ollama.ai/)は、様々なオープンソースLLMをローカルで実行するためのツールです。Mastraと組み合わせることで、完全にローカルな開発環境を構築できます。

まず、Ollamaをインストールして起動します：

```bash
# Ollamaのインストール（macOSの例）
curl -fsSL https://ollama.ai/install.sh | sh

# Ollamaサーバーの起動
ollama serve

# モデルのダウンロード（例：Llama 2）
ollama pull llama2
```

次に、Mastraプロジェクトで`@ai-sdk/ollama`パッケージをインストールします：

```bash
npm install @ai-sdk/ollama
```

そして、エージェント定義でOllamaモデルを使用するように設定します：

```typescript
import { Agent } from "@mastra/core/agent";
import { ollama } from "@ai-sdk/ollama";

export const localAgent = new Agent({
  name: "ローカルアシスタント",
  instructions: `あなたは親切で役立つアシスタントです。
ユーザーの質問に簡潔かつ正確に回答してください。`,
  model: ollama("llama2"),
});
```

これで、インターネット接続なしでもエージェントを開発・テストできるようになります。

## ローカル開発のベストプラクティス 🌟

Mastraを使ったローカル開発を効率的に行うためのベストプラクティスをいくつか紹介します：

### 1. 段階的な開発アプローチ

複雑なエージェントを一度に作るのではなく、基本機能から始めて徐々に機能を追加していくアプローチが効果的です：

1. まずは基本的な会話能力を持つシンプルなエージェントを作る
2. 一つずつツールを追加して、エージェントの能力を拡張する
3. エージェントの指示（instructions）を徐々に洗練させる
4. ユーザーフィードバックを取り入れて継続的に改善する

### 2. モックデータの活用

開発初期段階では、実際のAPIを呼び出す代わりにモックデータを使うことで、開発速度を上げることができます：

```typescript
// 天気APIのモック
const getWeatherMock = {
  // ...前述のツール定義...
  execute: async ({ city }) => {
    // 実際のAPI呼び出しの代わりにモックデータを返す
    const mockData = {
      "東京": { temperature: 22, condition: "晴れ" },
      "大阪": { temperature: 24, condition: "曇り" },
      "札幌": { temperature: 15, condition: "雨" },
      // デフォルト値
      "default": { temperature: 20, condition: "不明" }
    };
    
    return mockData[city] || mockData["default"];
  }
};
```

### 3. 環境ごとの設定分離

開発環境、テスト環境、本番環境で異なる設定を使い分けるために、環境変数を活用しましょう：

```typescript
// 環境に応じてモデルを切り替える
const modelProvider = process.env.NODE_ENV === 'production'
  ? openai("gpt-4o")
  : process.env.NODE_ENV === 'test'
    ? openai("gpt-3.5-turbo")
    : ollama("llama2");  // 開発環境ではローカルモデル

export const myAgent = new Agent({
  // ...
  model: modelProvider,
});
```

### 4. エラーハンドリングの実装

実際のアプリケーションでは、APIの障害やレート制限などの問題に対処するためのエラーハンドリングが重要です：

```typescript
const getWeatherWithErrorHandling = {
  // ...前述のツール定義...
  execute: async ({ city }) => {
    try {
      const response = await fetch(`https://weather-api.example.com/current?city=${city}`);
      
      if (!response.ok) {
        if (response.status === 429) {
          return { error: "レート制限に達しました。しばらく待ってから再試行してください。" };
        }
        return { error: `APIエラー: ${response.status}` };
      }
      
      return await response.json();
    } catch (error) {
      console.error("天気情報の取得に失敗:", error);
      return { error: "天気情報を取得できませんでした。ネットワーク接続を確認してください。" };
    }
  }
};
```

## ローカル開発から本番環境へ 🚢

ローカル環境での開発が完了したら、エージェントを本番環境にデプロイする準備が整います。Mastraは、様々なデプロイオプションをサポートしています：

### ビルドプロセス

本番環境用にプロジェクトをビルドするには、以下のコマンドを使います：

```bash
# プロジェクトをビルド
mastra build
```

このコマンドは、TypeScriptコードをJavaScriptにコンパイルし、必要なアセットを最適化します。ビルド結果は`dist`ディレクトリに出力されます。

### デプロイオプション

Mastraエージェントは、様々なプラットフォームにデプロイできます：

1. **Vercel**: サーバーレスデプロイに最適
   ```bash
   vercel deploy
   ```

2. **AWS Lambda**: サーバーレス関数として実行
   ```bash
   mastra deploy aws
   ```

3. **Docker**: コンテナ化してどこでも実行
   ```bash
   # Dockerfileが自動生成されます
   mastra generate dockerfile
   # イメージをビルドして実行
   docker build -t my-agent .
   docker run -p 3000:3000 my-agent
   ```

4. **Node.js サーバー**: 従来のサーバーとして実行
   ```bash
   # 本番モードで起動
   NODE_ENV=production node dist/index.js
   ```

## まとめ：ローカル開発の力 💪

この章では、Mastraを使ったローカル開発環境の構築方法を学びました。ローカル開発環境を活用することで、以下のメリットが得られます：

- 迅速な開発サイクルによる生産性の向上
- APIコスト削減とプライバシー保護
- 柔軟な実験と迅速なイテレーション
- ローカルモデルを活用したオフライン開発

Mastra CLIの強力な機能を使いこなすことで、アイデアを素早くプロトタイプ化し、高品質なAIエージェントを効率的に開発できるようになります。次の章では、これらのエージェントを本番環境にデプロイする方法について詳しく見ていきましょう。

ローカル開発環境は、あなたのアイデアを形にするための実験室です。思う存分試行錯誤して、革新的なAIエージェントを作り上げてください！🚀 