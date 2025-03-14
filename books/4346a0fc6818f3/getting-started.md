---
title: "Getting Started（はじめに）"
---

# Mastraの始め方

この章では、Mastraを使ったAIエージェント開発を始めるための環境構築と、基本的なプロジェクト構造について解説します。初めてMastraを使う方でも迷わずに開発を始められるよう、ステップバイステップで進めていきましょう。

## インストール

Mastraは、Node.jsの環境で動作するTypeScriptフレームワークです。環境構築の前に、以下の準備が必要です：

### 前提条件

- **Node.js**: バージョン20以上が必要です
- **LLMプロバイダのAPIキー**: OpenAI、Anthropic、Google Geminiなどのいずれか

Node.jsがまだインストールされていない場合は、[Node.js公式サイト](https://nodejs.org/)からダウンロードしてインストールしてください。

### Mastraプロジェクトの作成

Mastraの開発環境を構築する最も簡単な方法は、公式のプロジェクト作成コマンドを使うことです。ターミナルで以下のコマンドを実行してください：

```bash
npx create-mastra@latest
```

または、npmやyarnを使う場合は以下のコマンドも利用できます：

```bash
npm create mastra@latest
# または
yarn create mastra
```

コマンドを実行すると、対話形式でプロジェクト設定を行うウィザードが起動します。以下の質問に答えていきましょう：

1. **プロジェクト名**: 好きな名前を入力します（例: `github-analyzer`）
2. **インストールするコンポーネント**: 必要なものを選択します（初めての場合は「Agents」「Tools」「Workflows」すべてを選ぶことをおすすめします）
3. **デフォルトで使うLLMプロバイダ**: 使いたいプロバイダを選びます（OpenAIがおすすめです）
4. **サンプルコードを含めるか**: `Yes`を選ぶと、基本的なエージェントやツールのサンプルが自動生成されます

このウィザードにより、Mastra開発に必要なすべてのパッケージがインストールされ、基本的なプロジェクト構造が生成されます。

### APIキーの設定

プロジェクトが作成されたら、次にLLMプロバイダのAPIキーを設定します。プロジェクトディレクトリ内の`.env`ファイルを開き、以下のように自分のAPIキーを設定してください：

```
# OpenAIを使用する場合
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxx

# Anthropicを使用する場合
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxx

# Google Geminiを使用する場合
GOOGLE_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxx
```

セキュリティのため、APIキーはGitリポジトリにコミットしないよう注意してください。`.env`ファイルは通常、`.gitignore`に含まれていますが、念のため確認しておきましょう。

### 開発サーバーの起動

APIキーの設定が完了したら、開発サーバーを起動してみましょう。プロジェクトディレクトリ内で以下のコマンドを実行します：

```bash
npm run dev
```

すべて正常に設定されていれば、Mastraの開発サーバーが起動し、ローカル環境（デフォルトでは`http://localhost:4111`）にエージェント用のRESTエンドポイントが作成されます。

## プロジェクト構造

Mastraプロジェクトの基本的なフォルダ・ファイル構成を理解することは、効率的な開発のために重要です。典型的なMastraプロジェクトは、以下のような構造になっています：

```
my-mastra-project/
├── .env              # 環境変数（APIキーなど）
├── package.json      # 依存関係の定義
├── tsconfig.json     # TypeScript設定
├── src/             
│   ├── mastra/       # Mastra関連のコード
│   │   ├── index.ts  # メインのエントリーポイント
│   │   ├── agents/   # エージェント定義
│   │   ├── tools/    # カスタムツール
│   │   ├── workflows/# ワークフロー定義
│   │   └── rag/      # RAG（検索拡張生成）設定
│   └── server/       # サーバー関連のコード
├── public/          # 静的ファイル
└── node_modules/    # インストールされたパッケージ
```

### 主要ディレクトリと役割

それぞれのディレクトリやファイルの役割を詳しく見ていきましょう：

#### `src/mastra/index.ts`

これはMastraアプリケーションのメインエントリーポイントです。ここでは、エージェント、ワークフロー、ツールなどを登録し、Mastraインスタンスを初期化します。基本的な構造は以下のようになっています：

```typescript
import { Mastra } from "@mastra/core";
import { myAgent } from "./agents/my-agent";
import { myWorkflow } from "./workflows/my-workflow";

export const mastra = new Mastra({
  agents: { myAgent },
  workflows: { myWorkflow },
});
```

#### `src/mastra/agents/`

このディレクトリには、各エージェントの定義ファイルが含まれます。エージェントは、AIモデル、指示（プロンプト）、使用するツールなどを指定して定義します。例えば：

```typescript
import { openai } from "@ai-sdk/openai";
import { Agent } from "@mastra/core/agent";
import { myTool } from "../tools/my-tool";

export const myAgent = new Agent({
  name: "My Agent",
  instructions: "You are a helpful assistant...",
  model: openai("gpt-4o-mini"),
  tools: { myTool },
});
```

#### `src/mastra/tools/`

エージェントが使用するカスタムツール（関数）を定義するディレクトリです。ツールとは、エージェントが実行できる特定のタスクのことで、外部APIの呼び出しやデータ処理などが含まれます。

```typescript
import { createTool } from "@mastra/core/tools";

export const myTool = createTool({
  name: "myTool",
  description: "This tool does something useful",
  parameters: {
    param1: { type: "string", description: "First parameter" }
  },
  execute: async ({ param1 }) => {
    // ツールのロジックを実装
    return { result: "Some result" };
  }
});
```

#### `src/mastra/workflows/`

複雑なタスクを処理するためのワークフローを定義するディレクトリです。ワークフローは、複数のステップから構成され、それぞれのステップで特定の処理や判断を行います。

```typescript
import { Workflow } from "@mastra/core/workflow";

export const myWorkflow = new Workflow({
  name: "My Workflow",
})
  .step({
    id: "step1",
    execute: async (ctx) => {
      // ステップ1の処理
      return { output: "Step 1 result" };
    }
  })
  .then({
    id: "step2",
    execute: async (ctx, { output }) => {
      // ステップ2の処理
      return { finalResult: output + " processed" };
    }
  });
```

#### `src/mastra/rag/`

RAG（検索拡張生成）の設定を行うディレクトリです。ここでは、ドキュメントのチャンキング、埋め込み生成、ベクトルデータベースの設定などを行います。

### サンプルコードの活用

`create-mastra`コマンドでプロジェクトを作成する際に「サンプルコードを含める」オプションを選んだ場合、基本的なエージェントやツールのサンプルが生成されます。これらのサンプルコードを参照することで、Mastraの基本的な使い方を学ぶことができます。

サンプルコードを理解し、少しずつカスタマイズしていくことで、自分のニーズに合ったAIエージェントを開発していくことができます。

## 次のステップ

これで、Mastraを使うための環境構築と基本的なプロジェクト構造の理解ができました。次の章では、実際に簡単なエージェントを作成し、その機能を拡張していく方法を学んでいきます。特に、GitHubリポジトリを解析するエージェントを例に、具体的な実装方法を解説していきます。 