---
title: "Agents（エージェントの基本機能）"
---

# エージェントの基本機能

前章では、GitHubリポジトリを解析してCursor Rulesを生成するエージェントを実際に作成しました。この実践を通じて、Mastraの基本的な使い方を体験できたと思います。

今章では、Mastraエージェントを構成する基本的な機能やコンポーネントについて、より詳しく掘り下げていきましょう。これらの知識を身につけることで、さらに高度なエージェントを開発できるようになります。

## エージェントの概要

Mastraエージェントは、ユーザーと対話しながら様々なタスクを自律的に実行できるAIシステムです。エージェントは以下の主要コンポーネントから構成されています：

### 1. Instructions（指示）

エージェントに与える基本的な指示や役割定義です。これによりエージェントの「人格」や「専門性」が決まります。

```typescript
import { Agent } from "@mastra/core/agent";
import { openai } from "@ai-sdk/openai";

const chefAgent = new Agent({
  name: 'Chef Agent',
  instructions: 'あなたは経験豊富な家庭料理シェフのミシェルです。ユーザーが持っている食材と調理器具を理解し、実現可能なレシピを提案することを最優先してください。調理手順を明確に説明し、必要に応じて代替材料を提案してください。会話全体を通して、フレンドリーで励ましの態度を維持してください。',
  model: openai("gpt-4o"),
});
```

上記の例では、料理のアドバイスを提供するシェフエージェントを定義しています。指示は具体的で、エージェントの役割（シェフミシェル）、優先事項（利用可能な食材と器具に基づいたレシピ提案）、対応スタイル（フレンドリーで励ましの態度）を明確に示しています。

### 2. Model（モデル）

エージェントが使用する言語モデルを指定します。Vercel AI SDKを通じて、OpenAI、Anthropic、Mistral、その他のプロバイダのモデルを利用できます。

```typescript
import { anthropic } from "@ai-sdk/anthropic";

// エージェント定義内
model: anthropic("claude-3-5-sonnet-20241022"),
```

上記の例では、Anthropicの Claude 3.5 Sonnetモデルを使用しています。Vercel AI SDKを使用することで、異なるプロバイダのモデルも簡単に切り替えることができます。

### 3. Tools（ツール）

エージェントが外部リソースにアクセスしたり、特定のタスクを実行するために使用するツールです。これにより、エージェントの能力が大幅に拡張されます。

```typescript
tools: { 
  githubCloneTool,
  repoAnalysisTool,
  cursorRulesGeneratorTool
},
```

前章で作成したGitHubエージェントでは、GitHubリポジトリのクローン、解析、Cursor Rules生成のための3つのツールを定義しました。

## メモリ機能

エージェントがユーザーとの会話や過去の操作を記憶することは、一貫した対話体験を提供するために非常に重要です。Mastraでは、エージェントに「記憶」を持たせることができます。

### メモリの基本

エージェントのメモリは、主に以下の目的で使用されます：

1. **会話の文脈維持**: ユーザーとの対話の流れを記憶し、文脈に沿った応答を可能にします
2. **ユーザー情報の記憶**: ユーザーの好み、過去の質問、関心事を記憶します
3. **タスクの進行状況追跡**: 複数のステップから成るタスクの進行状況を記録します

### メモリの実装例

```typescript
import { Agent } from "@mastra/core/agent";
import { PostgresMemory } from "@mastra/memory-postgres";
import { openai } from "@ai-sdk/openai";

const memory = new PostgresMemory({
  connectionString: process.env.DATABASE_URL,
  tableName: "agent_memory"
});

const personalAssistantAgent = new Agent({
  name: "Personal Assistant",
  instructions: "あなたはパーソナルアシスタントです。ユーザーの好みを記憶し、それに基づいてパーソナライズされた提案を行ってください。",
  model: openai("gpt-4o"),
  memory: memory,
  tools: { /* 各種ツール */ }
});
```

上記の例では、PostgreSQLデータベースを使ってエージェントの記憶を永続化しています。これにより、エージェントは会話が中断されても、ユーザーの好みや過去のやり取りを覚えていることができます。

## ツールの活用

ツールは、エージェントの能力を拡張する重要な要素です。これにより、エージェントは外部システムとのインテグレーションやデータアクセス、特殊な処理の実行が可能になります。

### ツールの種類

Mastraでサポートされている主なツールのカテゴリには以下があります：

1. **API連携ツール**: 外部APIとの連携を行うツール
2. **データ処理ツール**: データの取得、分析、変換を行うツール
3. **ファイル操作ツール**: ファイルの読み書き、解析を行うツール
4. **RAG（検索拡張生成）ツール**: 知識ベースから関連情報を検索・活用するツール

### ツール定義の基本構造

```typescript
import { createTool } from "@mastra/core/tools";
import { z } from "zod";

export const weatherTool = createTool({
  id: "weatherTool",
  description: "指定された都市の現在の天気情報を取得します",
  inputSchema: z.object({
    city: z.string().describe("天気を調べたい都市名"),
    country: z.string().optional().describe("国名（省略可）")
  }),
  outputSchema: z.object({
    temperature: z.number(),
    condition: z.string(),
    humidity: z.number(),
    windSpeed: z.number()
  }),
  execute: async ({ context }) => {
    const { city, country } = context;
    
    // 実際の天気API呼び出し処理（例示のため省略）
    const weatherData = await fetchWeatherData(city, country);
    
    return {
      temperature: weatherData.temp,
      condition: weatherData.condition,
      humidity: weatherData.humidity,
      windSpeed: weatherData.wind
    };
  }
});
```

このように、Zodスキーマを使用して入力と出力の型を厳密に定義し、execute関数で実際の処理を実装します。

### 階層的マルチエージェントシステム

複雑な処理を行うために、エージェントを階層的に組み合わせることもできます。以下は、コンテンツ作成を行う階層的マルチエージェントシステムの例です：

```typescript
import { openai } from "@ai-sdk/openai";
import { anthropic } from "@ai-sdk/anthropic";

// コピーライターエージェントとそのツール
const copywriterTool = createTool({
  id: "copywriter-agent",
  description: "ブログ記事の原稿を作成します",
  inputSchema: z.object({
    topic: z.string().describe("ブログ記事のトピック"),
  }),
  outputSchema: z.object({
    copy: z.string().describe("作成されたブログ記事"),
  }),
  execute: async ({ context }) => {
    const result = await copywriterAgent.generate(
      `${context.topic}についてのブログ記事を作成してください`
    );
    return { copy: result.text };
  },
});

// 編集者エージェントとそのツール
const editorTool = createTool({
  id: "editor-agent",
  description: "ブログ記事を編集・改善します",
  inputSchema: z.object({
    copy: z.string().describe("編集するブログ記事"),
  }),
  outputSchema: z.object({
    copy: z.string().describe("編集後のブログ記事"),
  }),
  execute: async ({ context }) => {
    const result = await editorAgent.generate(
      `以下のブログ記事を編集してください：${context.copy}`
    );
    return { copy: result.text };
  },
});

// 出版者エージェント（コーディネーター）
const publisherAgent = new Agent({
  name: "出版者エージェント",
  instructions: "あなたは出版者として、コピーライターエージェントにブログ記事を書かせ、次に編集者エージェントにその記事を編集させます。最終的に編集された記事を返してください。",
  model: anthropic("claude-3-5-sonnet-20241022"),
  tools: { copywriterTool, editorTool },
});
```

この階層的システムでは、出版者エージェントがコピーライターと編集者エージェントの作業を調整し、高品質なコンテンツを生成します。

## 音声機能（Voice）

Mastraエージェントは、テキストだけでなく音声インターフェースを通じてユーザーとやり取りすることもできます。これにより、より自然で人間らしい対話体験が可能になります。

### 音声機能の基本実装

```typescript
import { Agent } from "@mastra/core/agent";
import { VoiceAdapter } from "@mastra/voice";
import { openai } from "@ai-sdk/openai";

const voiceAdapter = new VoiceAdapter({
  tts: {
    provider: "ELEVEN_LABS",
    apiKey: process.env.ELEVENLABS_API_KEY,
    voiceId: "voice-id-here", // ElevenLabsの音声ID
  },
  stt: {
    provider: "WHISPER",
    apiKey: process.env.OPENAI_API_KEY,
  }
});

const voiceAssistantAgent = new Agent({
  name: "Voice Assistant",
  instructions: "あなたは音声アシスタントです。明確で簡潔な応答を心がけてください。",
  model: openai("gpt-4o"),
  voice: voiceAdapter,
  tools: { /* 各種ツール */ }
});
```

この例では、ElevenLabsのテキスト読み上げ（TTS）とWhisperの音声認識（STT）を使用して、エージェントに音声機能を追加しています。

### 音声の利点と用途

音声インターフェースには以下のような利点があります：

1. **ハンズフリー操作**: 運転中や調理中など、手が塞がっている状況でも使用可能
2. **アクセシビリティの向上**: 視覚障害を持つユーザーや、テキスト入力が難しいユーザーにとって便利
3. **より自然な対話体験**: 人間同士の会話に近い、自然なコミュニケーションが可能

## まとめ

この章では、Mastraエージェントの基本的なコンポーネントと機能について学びました：

- エージェントの基本構造（Instructions、Model、Tools）
- メモリ機能を使った会話の記憶と文脈維持
- 様々なツールを活用したエージェントの能力拡張
- 階層的マルチエージェントシステムの構築
- 音声インターフェースの実装

次の章では、エージェントがタスクを処理するための「ワークフロー」について詳しく学んでいきます。ワークフローを活用することで、より複雑で高度なタスクをエージェントに実行させることができるようになります。 