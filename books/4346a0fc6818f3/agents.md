---
title: "Agents（エージェントの基本機能）"
---

# エージェントの基本機能の冒険へようこそ！ 🚀

前章では、GitHubリポジトリを解析してCursor Rulesを生成するエージェントを一緒に作り上げました。実際に手を動かして、Mastraの魔法を体験できたのではないでしょうか？素晴らしい第一歩です！

さあ、今章ではさらに冒険を進めて、Mastraエージェントを構成する基本的な機能やコンポーネントの秘密を解き明かしていきましょう。まるで魔法の杖の仕組みを学ぶように、エージェントの内側に潜む驚きの仕掛けを一緒に探検します。この知識があれば、あなたも魔法使いのように、さらに強力なエージェントを自在に操れるようになりますよ！✨

## エージェントの概要 - 魔法の仕組み 🧙‍♂️

Mastraエージェントは、ユーザーと対話しながらさまざまなタスクを自律的に実行できる、まるで頼れるパートナーのようなAIシステムです。エージェントの体には、どんな魔法の部品が組み込まれているのでしょうか？ここでその秘密を明かします！

### 1. Instructions（指示）- エージェントの心臓部 💗

エージェントに与える基本的な指示や役割定義は、まるでエージェントの「人格」や「専門性」を形作る心臓部のようなものです。ここで設定した性格や役割によって、エージェントの振る舞いが大きく変わってきます。

```typescript
import { Agent } from "@mastra/core/agent";
import { openai } from "@ai-sdk/openai";

const chefAgent = new Agent({
  name: 'Chef Agent',
  instructions: 'あなたは経験豊富な家庭料理シェフのミシェルです。ユーザーが持っている食材と調理器具を理解し、実現可能なレシピを提案することを最優先してください。調理手順を明確に説明し、必要に応じて代替材料を提案してください。会話全体を通して、フレンドリーで励ましの態度を維持してください。',
  model: openai("gpt-4o"),
});
```

この例を見てください！ちょっとしたコード魔法で、料理のエキスパートを生み出しています。指示は具体的かつ明確で、エージェントの「シェフミシェル」としての役割、優先事項（利用可能な食材と器具に基づいたレシピ提案）、対応スタイル（フレンドリーで励ましの態度）をしっかり定義しています。シェフミシェルは、まるで本物のシェフのように、あなたの冷蔵庫にある材料で最高の料理を提案してくれるでしょう！🍳

### 2. Model（モデル）- エージェントの頭脳 🧠

エージェントが使用する言語モデルは、まさにエージェントの「頭脳」です。Vercel AI SDKの魔法により、OpenAI、Anthropic、Mistral、その他のプロバイダが提供する最強のAIモデルを簡単に利用できます。まるで異なる能力を持つ魔法の杖を選ぶような感覚ですね！

```typescript
import { anthropic } from "@ai-sdk/anthropic";

// エージェント定義内
model: anthropic("claude-3-5-sonnet-20241022"),
```

たった1行のコードで、Anthropicの最新鋭モデルであるClaude 3.5 Sonnetを呼び出しています。素晴らしくないですか？もし別のモデルを試したくなったら、この1行を変えるだけで、エージェントは全く異なる思考パターンを持つことができます。まるで魔法の呪文を変えるだけで、違う能力が発現するようなものです！✨

### 3. Tools（ツール）- エージェントの手足 🛠️

ツールは、エージェントが世界と関わるための「手足」です。これらのツールにより、エージェントはあなたの代わりに様々なアクションを実行できるようになります。前章で作ったGitHubエージェントを思い出してください：

```typescript
tools: { 
  githubCloneTool,
  repoAnalysisTool,
  cursorRulesGeneratorTool
},
```

このコードで、エージェントに3つの魔法の道具を与えました。これらの道具があるからこそ、エージェントはGitHubのリポジトリをクローンし、解析し、Cursor Rulesを生成するという驚くべき能力を発揮できるのです。適切なツールがあれば、エージェントはほぼどんなことでもできるようになります。可能性は無限大です！🌟

## メモリ機能 - エージェントの記憶の仕組み 🧠💭

会話の途中で「さっき言ったあのこと、覚えてる？」と聞かれて、「何のことですか？」と返されたら残念ですよね。安心してください！Mastraエージェントには優れた「記憶力」を持たせることができます。まるで親友との会話のように、文脈を理解し続けてくれるんです。

### メモリの基本 - 忘れない魔法 ✨

エージェントのメモリは、以下のような素晴らしい力を発揮します：

1. **会話の文脈維持** 📝: 「それについてもう少し教えて」と言われても、「それ」が何かをちゃんと覚えています
2. **ユーザー情報の記憶** 👤: あなたの好みや過去の質問、関心事をしっかり記憶。「いつもの」と言えば理解してくれます
3. **タスクの進行状況追跡** ⏱️: 「続きをやろう」と言えば、前回どこまで進んだかを覚えています

ちょっと想像してみてください。長期間にわたって、あなたの好みや過去の会話を覚えているアシスタント...便利すぎませんか？😲

### メモリの実装例 - 記憶の魔法を使ってみよう！

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

このコードを見てください！PostgreSQLデータベースを使って、エージェントの記憶を永続化しています。これにより、たとえサーバーを再起動しても、エージェントはあなたとの過去のやり取りを鮮明に覚えています。「前回のコーヒーショップの名前は？」と聞けば、即座に答えてくれるでしょう。まるで魔法のような記憶力ですね！🔮

## ツールの活用 - エージェントの能力拡張 🧰

ツールは、エージェントにとって魔法の杖のようなもの。これらを使いこなすことで、エージェントは驚くべき能力を身につけます。キッチンの調理器具が増えるほどに作れる料理のバリエーションが増えるように、ツールが増えるほどエージェントの可能性も広がっていきます！

### ツールの種類 - 魔法の道具箱 🧙‍♀️

Mastraで使える魔法の道具には、様々な種類があります：

1. **API連携ツール** 🌐: 外部の世界（API）と繋がり、情報を取得したり操作したりできます
2. **データ処理ツール** 📊: 膨大なデータを理解し、分析し、意味ある形に変換します
3. **ファイル操作ツール** 📂: ドキュメントを読み書きし、情報を整理します
4. **RAG（検索拡張生成）ツール** 🔍: 膨大な知識の海から、必要な情報だけを釣り上げます

これらの道具を組み合わせれば、エージェントは何でもできるスーパーヒーローのようになります！🦸‍♂️

### ツール定義の基本構造 - 道具の作り方

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

このコードは少し長く見えるかもしれませんが、心配しないでください！実はとてもシンプルなレシピに従っています：

1. ツールに名前と説明を付ける
2. 入力と出力の形を明確に定義する（Zodの魔法を使って）
3. 実行する処理を書く

こうして作られたツールは、エージェントにとって信頼できる助手となり、天気を調べるという特殊能力を与えてくれます！🌤️

### 階層的マルチエージェントシステム - チームで働く魔法使い 👥

一人の魔法使いも強力ですが、専門分野の異なる魔法使いたちがチームを組んだら？もっと驚くべきことができますよね！Mastraでは、複数のエージェントを連携させて、より複雑な問題を解決できます：

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

このコードでは、まるで小さな出版社を作っているようです！コピーライター、編集者、そして彼らをまとめる出版者がチームとなって、高品質なブログ記事を作り出します。一人ではできないことも、チームならできる—そんな魔法のような連携プレイです！👨‍👩‍👧‍👦

## 音声機能（Voice）- エージェントに声を与える 🎤

文字だけのコミュニケーションも素晴らしいですが、人間同士のように声で会話できたら、もっと自然でしょう？そう、Mastraエージェントに「声」を与えることもできるんです！

### 音声機能の基本実装 - 声の魔法

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

このコードでは、エージェントに魔法の耳と口を与えています！ElevenLabsの自然な音声合成（TTS）でエージェントに声を与え、Whisperの優れた音声認識（STT）であなたの言葉を理解させます。まるで映画に出てくるAIアシスタントのような体験が、このコードで実現できるんです！🔊

### 音声の利点と用途 - 声の魔法がもたらす可能性

音声インターフェースには、たくさんの魔法のような利点があります：

1. **ハンズフリー操作** 👐: 料理中や運転中でも、手を使わずに操作できます
2. **アクセシビリティの向上** ♿: 視覚障害をお持ちの方や、テキスト入力が難しい方でも簡単に利用できます
3. **より自然な対話体験** 🗣️: まるで友人と話すような、温かみのあるコミュニケーションが可能になります

想像してみてください—朝起きて「今日の予定は？」と話しかけるだけで、あなたの一日のスケジュールを読み上げてくれるアシスタント。未来の技術ではなく、今すぐ実現できる魔法なんです！✨

## まとめ - 魔法の旅を振り返って 🌈

おめでとうございます！この章では、Mastraエージェントを構成する魔法の部品と機能について学びました：

- エージェントの基本構造（Instructions、Model、Tools）—エージェントの心臓部、頭脳、手足
- メモリ機能—エージェントに人間らしい記憶力を与える魔法
- ツールの活用—エージェントの能力を無限に拡張する魔法の道具箱
- 階層的マルチエージェントシステム—魔法使いたちのチームワーク
- 音声インターフェース—エージェントに声を与える魔法

ここまで来たあなたは、もはやMastraの見習い魔法使いではなく、立派な魔法使いへと成長しています！🧙‍♂️

次の章では、「ワークフロー」という、さらに高度な魔法について学んでいきます。エージェントがより複雑なタスクを自律的にこなせるようになる、ワークフローの秘密をぜひ一緒に探検しましょう！

さあ、次の冒険へ出発しましょう！🚀 