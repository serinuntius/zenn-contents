---
title: "第11章：エージェントの基本機能 (執筆中 🖋️)"
---

:::message alert
**🚧 本書は現在執筆中です 🚧**

読者の皆様へ：本書は現在執筆進行中の状態でリリースしています。早く皆様にお届けしたいという思いから、一部未完成の章も含めて公開しております。今後定期的に内容が更新・拡充されますので、ご了承ください。最新のアップデートをお楽しみに！
:::

# エージェントの基本機能を探検しよう！ 🚀

前章では、GitHubリポジトリを解析してCursor Rulesを生成するエージェントを一緒に作り上げました。実際に手を動かして、Mastraの機能を体験できたのではないでしょうか？素晴らしい第一歩です！

さあ、今章ではさらに探索を進めて、Mastraエージェントを構成する基本的な機能やコンポーネントの詳細を解き明かしていきましょう。エージェントの内側に潜む驚きの仕組みを一緒に探検します。この知識があれば、あなたもさらに強力なエージェントを自在に操れるようになりますよ！✨

## エージェントの概要 - 基本の仕組み 🧠

Mastraエージェントは、ユーザーと対話しながらさまざまなタスクを自律的に実行できる、まるで頼れるパートナーのようなAIシステムです。エージェントの内部には、どんな重要な構成要素が組み込まれているのでしょうか？ここでその秘密を明かします！

### 1. Instructions（指示）- エージェントの心臓部 💗

エージェントに与える基本的な指示や役割定義は、まるでエージェントの「人格」や「専門性」を形作る心臓部のようなものです。ここで設定した性格や役割によって、エージェントの振る舞いが大きく変わってきます。

```typescript
import { Agent } from "@mastra/core/agent";
import { createGoogleGenerativeAI } from "@ai-sdk/google";
import { anthropic } from "@ai-sdk/anthropic";

// Google Gemini AIプロバイダーの作成
const google = createGoogleGenerativeAI({
    apiKey: process.env.GOOGLE_API_KEY || "",
});

// Geminiモデルのインスタンスを作成
const geminiModel = google("gemini-2.0-flash-001");

const chefAgent = new Agent({
  name: 'Chef Agent',
  instructions: 'あなたは経験豊富な家庭料理シェフのミシェルです。ユーザーが持っている食材と調理器具を理解し、実現可能なレシピを提案することを最優先してください。調理手順を明確に説明し、必要に応じて代替材料を提案してください。会話全体を通して、フレンドリーで励ましの態度を維持してください。',
  model: geminiModel,
  memory: {
    type: 'buffer',
    maxMessages: 10
  }
});
```

このコードを見てください！GoogleのGemini Flash 2.0モデルを使って、エージェントの記憶力を実装しています。これにより、エージェントはユーザーの好みを記憶し、将来の会話でそれを思い出すことができます。この記憶力は、エージェントがより自然で親しみやすいコミュニケーションを提供するのに役立ちます。🔮

### 2. Model（モデル）- エージェントの頭脳 🧠

エージェントが使用する言語モデルは、まさにエージェントの「頭脳」です。Vercel AI SDKのおかげで、Google、Anthropic、OpenAI、その他のプロバイダが提供するAIモデルを簡単に利用できます。特に、Googleのジェミニモデルはコストパフォーマンスと速度のバランスが優れていてお勧めです！

```typescript
import { createGoogleGenerativeAI } from "@ai-sdk/google";
import { anthropic } from "@ai-sdk/anthropic";

// Google Gemini AIプロバイダーの初期化
const google = createGoogleGenerativeAI({
    apiKey: process.env.GOOGLE_API_KEY || "",
});

// 最もコスパの良いジェミニモデル
const geminiModel = google("gemini-2.0-flash-001");

// または他のプロバイダも使用可能
const claudeModel = anthropic("claude-3-5-sonnet-20241022");
```

たった数行のコードで、GoogleのGemini Flash 2.0モデルを呼び出せます。このモデルは処理速度、コスト、性能のバランスが非常に優れており、多くのユースケースに最適です。もちろん、必要に応じて他のモデルも利用できますが、まずはジェミニから始めることをお勧めします！✨

### 3. Tools（ツール）- エージェントの手足 🛠️

ツールは、エージェントが世界と関わるための「手足」です。これらのツールにより、エージェントはあなたの代わりに様々なアクションを実行できるようになります。前章で作ったGitHubエージェントを思い出してください：

```typescript
tools: { 
  githubCloneTool,
  repoAnalysisTool,
  cursorRulesGeneratorTool
},
```

このコードで、エージェントに3つの強力なツールを与えました。これらのツールがあるからこそ、エージェントはGitHubのリポジトリをクローンし、解析し、Cursor Rulesを生成するという驚くべき能力を発揮できるのです。適切なツールがあれば、エージェントはほぼどんなことでもできるようになります。可能性は無限大です！🌟

## メモリ機能 - エージェントの記憶の仕組み 🧠💭

会話の途中で「さっき言ったあのこと、覚えてる？」と聞かれて、「何のことですか？」と返されたら残念ですよね。安心してください！Mastraエージェントには優れた「記憶力」を持たせることができます。まるで親友との会話のように、文脈を理解し続けてくれるんです。

### メモリの基本 - 忘れない技術 ✨

エージェントのメモリは、以下のような素晴らしい力を発揮します：

1. **会話の文脈維持** 📝: 「それについてもう少し教えて」と言われても、「それ」が何かをちゃんと覚えています
2. **ユーザー情報の記憶** 👤: あなたの好みや過去の質問、関心事をしっかり記憶。「いつもの」と言えば理解してくれます
3. **タスクの進行状況追跡** ⏱️: 「続きをやろう」と言えば、前回どこまで進んだかを覚えています

ちょっと想像してみてください。長期間にわたって、あなたの好みや過去の会話を覚えているアシスタント...便利すぎませんか？😲

### メモリの実装例 - 記憶機能を実装しよう！

```typescript
import { Agent } from "@mastra/core/agent";
import { createGoogleGenerativeAI } from "@ai-sdk/google";
import { anthropic } from "@ai-sdk/anthropic";
import { Memory } from "@mastra/memory";

// Google Gemini AIプロバイダーの作成
const google = createGoogleGenerativeAI({
    apiKey: process.env.GOOGLE_API_KEY || "",
});

// Geminiモデルのインスタンスを作成
const geminiModel = google("gemini-2.0-flash-001");

// メモリ設定
const memory = new Memory({
  options: {
    lastMessages: 10, // 以前のmaxMessagesに相当
    workingMemory: {
      enabled: true // ワーキングメモリを有効化
    }
  }
});

const memoryAgent = new Agent({
  name: 'Memory Agent',
  instructions: 'あなたはユーザーの好みを記憶するアシスタントです。ユーザーの好みについて言及されたら（好きな色、食べ物、映画など）、それをメモリに保存し、将来の会話で思い出してください。',
  model: geminiModel,
  memory
});
```

このコードを見てください！GoogleのGemini Flash 2.0モデルを使って、エージェントの記憶力を実装しています。これにより、エージェントはユーザーの好みを記憶し、将来の会話でそれを思い出すことができます。この記憶力は、エージェントがより自然で親しみやすいコミュニケーションを提供するのに役立ちます。🔮

## ツールの活用 - エージェントの能力拡張 🧰

ツールは、エージェントにとって重要な能力拡張装置です。これらを使いこなすことで、エージェントは驚くべき能力を身につけます。キッチンの調理器具が増えるほどに作れる料理のバリエーションが増えるように、ツールが増えるほどエージェントの可能性も広がっていきます！

### ツールの種類 - 多様な道具セット 🛠️

Mastraで使える様々なツールには、以下のような種類があります：

1. **API連携ツール** 🌐: 外部の世界（API）と繋がり、情報を取得したり操作したりできます
2. **データ処理ツール** 📊: 膨大なデータを理解し、分析し、意味ある形に変換します
3. **ファイル操作ツール** 📂: ドキュメントを読み書きし、情報を整理します
4. **RAG（検索拡張生成）ツール** 🔍: 膨大な知識の中から、必要な情報だけを抽出します

これらの道具を組み合わせれば、エージェントは非常に高機能なアシスタントになります！🦸‍♂️

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
2. 入力と出力の形を明確に定義する（Zodの力を使って）
3. 実行する処理を書く

こうして作られたツールは、エージェントにとって信頼できる助手となり、天気を調べるという特別な能力を与えてくれます！🌤️

### 階層的マルチエージェントシステム - チームで働くエージェント 👥

一人のエージェントも強力ですが、専門分野の異なるエージェントたちがチームを組んだら？もっと驚くべきことができますよね！Mastraでは、複数のエージェントを連携させて、より複雑な問題を解決できます：

```typescript
import { createGoogleGenerativeAI } from "@ai-sdk/google";
import { anthropic } from "@ai-sdk/anthropic";
import { z } from "zod";
import { Agent } from "@mastra/core/agent";
import { createTool } from "@mastra/core/tools";

// Google Gemini AIプロバイダーの作成
const google = createGoogleGenerativeAI({
    apiKey: process.env.GOOGLE_API_KEY || "",
});

// Geminiモデルのインスタンスを作成
const geminiModel = google("gemini-2.0-flash-001");
// Claudeモデルのインスタンスを作成
const claudeModel = anthropic("claude-3-5-sonnet-20241022");

// コピーライターエージェント
const copywriterAgent = new Agent({
  name: "コピーライターエージェント",
  instructions: "あなたは優れたコピーライターです。魅力的でSEOに最適化されたブログ記事を書いてください。",
  model: geminiModel
});

// 編集者エージェント
const editorAgent = new Agent({
  name: "編集者エージェント",
  instructions: "あなたは厳格な編集者です。文法、スタイル、流れを改善し、コンテンツを洗練させてください。",
  model: geminiModel
});

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
  model: claudeModel,
  tools: { copywriterTool, editorTool },
});
```

このコードでは、まるで小さな出版社を作っているようです！コピーライター、編集者、そして彼らをまとめる出版者がチームとなって、高品質なブログ記事を作り出します。一人ではできないことも、チームならできる—そんな効果的な連携プレイです！👨‍👩‍👧‍👦

## 音声機能（Voice）- エージェントに声を与える 🎤

文字だけのコミュニケーションも素晴らしいですが、人間同士のように声で会話できたら、もっと自然でしょう？そう、Mastraエージェントに「声」を与えることもできるんです！

### 音声機能の基本実装 - 声の実装方法

```typescript
import { Agent } from "@mastra/core/agent";
import { VoiceAdapter } from "@mastra/voice";
import { createGoogleGenerativeAI } from "@ai-sdk/google";
import { anthropic } from "@ai-sdk/anthropic";

const google = createGoogleGenerativeAI({
    apiKey: process.env.GOOGLE_API_KEY || "",
});

// Geminiモデルのインスタンスを作成
const geminiModel = google("gemini-2.0-flash-001");

// Claudeモデルのインスタンスを作成
const claudeModel = anthropic("claude-3-5-sonnet-20241022");

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
  model: geminiModel,
  voice: voiceAdapter,
  tools: { /* 各種ツール */ }
});
```

このコードでは、エージェントに優れた耳と口を与えています！ElevenLabsの自然な音声合成（TTS）でエージェントに声を与え、Whisperの優れた音声認識（STT）であなたの言葉を理解させます。まるで映画に出てくるAIアシスタントのような体験が、このコードで実現できるんです！🔊

### 音声の利点と用途 - 声のインターフェースがもたらす可能性

音声インターフェースには、たくさんの素晴らしい利点があります：

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

ここまで来たあなたは、もはやMastraの見習い魔法使いではなく、立派な魔法使いへと成長しています！🧠

次の章では、「ワークフロー」という、さらに高度な魔法について学んでいきます。エージェントがより複雑なタスクを自律的にこなせるようになる、ワークフローの秘密をぜひ一緒に探検しましょう！

さあ、次の冒険へ出発しましょう！🚀 