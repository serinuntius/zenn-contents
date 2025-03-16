---
title: "第13章：RAG（検索拡張生成）"
--- 

# RAGの力を解き放つ 🔍✨

## RAGとは？LLMの知識強化システム 📚

RAG（Retrieval-Augmented Generation：検索拡張生成）とは、LLMの知識を、あなた独自のデータソースから取り出した情報で強化する特別な技術です！これは、AIモデルが自分自身の事前学習知識だけでなく、あなたが用意した特定の情報も参照して、より正確な回答を生成できるようにする仕組みなんです。✨

Mastraの RAGシステムは、以下の素晴らしい機能を提供します：

- 📜 ドキュメントを処理し、ベクトル埋め込みに変換する標準化されたAPI
- 🗄️ 様々なベクトルストアと連携できる柔軟性
- ✂️ ドキュメントを最適なチャンクに分割する効率的な戦略
- 📊 埋め込みと検索のパフォーマンスを可視化する評価ツール

「でも難しそう...」と思いましたか？心配いりません！RAGの技術を習得すれば、あなたのエージェントはより賢く、正確になり、最新の情報を使った回答ができるようになります。これはまるで、AIに最新の百科事典や専門書を与えるようなものです。結果として、「ふーん、そう思うだけかい？」という曖昧な回答から、「実はこれには確かな根拠があります！」という信頼性の高い回答へと進化するのです！🌟

## RAGの処理手順 🔄

RAGのプロセスは、次のような効率的な流れで実行されます：

1. **文書の収集** 📚: テキスト、HTML、マークダウンなどの形式から知識を集める
2. **情報の分割** ✂️: 大きな文書を理解しやすい小さな断片（チャンク）に分ける
3. **ベクトル変換** 💎: 断片をベクトル（数値配列）に変換する
4. **データの保存** 🗃️: ベクトルをベクトルデータベースに収める
5. **関連情報の検索** 🔍: 質問に関連する最も価値ある断片を呼び出す
6. **情報の整理** 📋: 必要に応じて情報を重要度順に並べ直す
7. **回答の生成** ✨: 集めた情報を使ってLLMが最適な回答を作成する

それでは、基本的なRAG実装のコードを見てみましょう：

```typescript
import { embedMany } from "ai";
import { openai } from "@ai-sdk/openai";
import { PgVector } from "@mastra/pg";
import { MDocument } from "@mastra/rag";

// 1. 文書を用意
const doc = MDocument.fromText(`ここに重要な情報を記入...`);

// 2. 文書を読みやすい断片に分割
const chunks = await doc.chunk({
  strategy: "recursive",
  size: 512,
  overlap: 50,
});

// 3. 断片をベクトルに変換
const { embeddings } = await embedMany({
  values: chunks,
  model: openai.embedding("text-embedding-3-small"),
});

// 4. ベクトルをデータベースに保存
const pgVector = new PgVector(process.env.POSTGRES_CONNECTION_STRING);
await pgVector.upsert({
  indexName: "embeddings",
  vectors: embeddings,
});

// 5. 関連する知識を検索
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: queryVector, // 質問のベクトル
  topK: 3,
});

console.log("見つけた関連情報:", results);
```

「うわぁ、難しそう...」と思ったかもしれませんね。でも大丈夫！これは単に「文書を集めて→分けて→数値化して→保存して→必要な時に取り出す」という、私たちが日常的に行っていることを、コンピュータ言語で表現しただけなんです。一歩ずつ進めていきましょう！🚶‍♀️

## 文書を整理する 📚✨

RAGの第一歩は、文書（ドキュメント）を集めることです。Mastraでは、`MDocument`というツールを使って、さまざまな形式のデータを取り込むことができます：

```typescript
const docFromText = MDocument.fromText("昔々あるところに...");
const docFromHTML = MDocument.fromHTML("<html>興味深い歴史...</html>");
const docFromMarkdown = MDocument.fromMarkdown("# 基本原則...");
const docFromJSON = MDocument.fromJSON(`{ "トピック": "重要な情報" }`);
```

これはまるで、図書館の司書が様々な形式の資料（テキスト、ウェブページ、マークダウン文書、データ集）を集めているようなものですね！📚

### 情報を小さく分ける技術 ✂️

大きな文書をそのまま処理するのは大変です。そこで、`chunk`という機能で文書を小さな断片に分けます。Mastraは様々な分け方を知っていて、文書の種類に合わせた最適な分割方法を選べます：

* `recursive` 🌲: 内容の構造を考慮した賢い分割（まるで目次から章、節へと分けるよう）
* `character` 📏: シンプルに文字数で区切る方法
* `token` 🔤: 言葉の意味の単位で区切る方法
* `markdown` 📝: マークダウン形式の文書に適した分割
* `html` 🌐: ウェブページの構造に合わせた分割
* `json` 📊: データ構造に合わせた分割
* `latex` 📑: 学術論文などの特殊な形式に合わせた分割

例えば、賢い分割方法（recursive）を使うにはこんなコードを書きます：

```typescript
const chunks = await doc.chunk({
  strategy: "recursive",  // 賢い分割方法を選択
  size: 512,              // だいたいこのくらいの大きさに
  overlap: 50,            // 少し重複させて、文脈を失わないように
  separator: "\n",        // 段落ごとに区切るのがいいかも
  extract: {
    metadata: true,       // 文書の情報（著者、日付など）も取っておく
  },
});
```

こうして、大きな文書が読みやすい小さなページに分かれました。これで処理するのも、必要な部分を探すのも簡単になります！🧠✨

## ベクトル変換プロセス 💫

さて、集めた情報の断片をベクトル（ベクトル埋め込み）に変換しましょう。これは言葉の意味を数値の配列に変換する、「言葉を数値データに変える」重要なプロセスです！

### OpenAIのモデルで変換する場合 🔮

```typescript
import { openai } from "@ai-sdk/openai";
import { embedMany } from "ai";

const { embeddings } = await embedMany({
  model: openai.embedding('text-embedding-3-small'),
  values: chunks.map(chunk => chunk.text),
});
```

### Cohereのモデルで変換する場合 🌈

```typescript
import { embedMany } from 'ai';
import { cohere } from '@ai-sdk/cohere';

const { embeddings } = await embedMany({
  model: cohere.embedding('embed-english-v3.0'),
  values: chunks.map(chunk => chunk.text),
});
```

この処理が生み出すのは、テキストの意味を表す数字の配列。人間には単なる数字の羅列に見えますが、コンピューターにとっては「ああ、これは猫についての文章で、これは天気についての文章だ」と意味を理解できる、重要なデータなのです。✨

## ベクトルデータベース 🏰

ベクトルに変換した情報は、特別なデータベース（ベクトルデータベース）に保管します。Mastraは様々なデータベースと連携できるので、あなたのプロジェクトに最適なものを選べます！

### 選べるベクトルデータベース 🗄️

- **PostgreSQL with pgvector** 🐘: すでにPostgreSQLを使っているプロジェクトに最適
- **Pinecone** 🌲: 大量のデータを高速に検索したい上級ユーザーに
- **Qdrant** ⚡: パワフルな検索機能が必要な時に
- **Chroma** 🌈: 誰でも使える汎用的なデータベース
- **Astra DB** 🌠: クラウドベースのデータベース
- **LibSQL** 📱: 小規模プロジェクトでも使える軽量なデータベース
- **Upstash** ☁️: サーバーレス環境での利用に
- **Cloudflare Vectorize** 🌐: 世界中どこでも高速アクセスできるデータベース

例えば、PostgreSQLというデータベースを使うなら、こんなコードを書きます：

```typescript
import { PgVector } from '@mastra/pg';

const store = new PgVector(process.env.POSTGRES_CONNECTION_STRING);

// データベースのインデックスを準備
await store.createIndex({
  indexName: "my-collection",
  dimension: 1536,  // ベクトルの次元数
});

// データを保存
await store.upsert({
  indexName: "my-collection",
  vectors: embeddings,
  metadata: chunks.map(chunk => ({
    text: chunk.text,
    // 整理するための情報
    source: chunk.source,
    category: chunk.category,
    createdAt: new Date().toISOString(),
  })),
});
```

### メタデータの活用 📋

ベクトル化した情報には、「どの文書から来たか」「いつの情報か」といった補足情報（メタデータ）をつけておくと、後で探しやすくなります。これは図書館で本に「分類ラベル」や「出版日」を記録するようなものです：

```typescript
await vectorStore.upsert({
  indexName: "embeddings",
  vectors: embeddings,
  metadata: chunks.map((chunk) => ({
    // 基本情報
    text: chunk.text,
    id: chunk.id,
    // 文書の整理情報
    source: chunk.source,
    category: chunk.category,
    // 時間の記録
    createdAt: new Date().toISOString(),
    version: "1.0",
    // その他の特徴
    language: chunk.language,
    author: chunk.author,
  })),
});
```

これでデータベースに、きちんと整理された情報が保管されました。「特定のトピックに関する情報だけ」「去年書かれた情報だけ」などと指定して検索できるようになります！📚✨

## 保存した情報を活用する 🔍

さあ、保管した情報を実際に使ってみましょう！質問が来たら、関連する情報を呼び出して、最適な回答を作る作業の始まりです。

### 基本的な情報検索 🧠

最もシンプルな方法は、質問を同じくベクトルに変換して、似たベクトルを持つ情報を探す方法です：

```typescript
import { openai } from "@ai-sdk/openai";
import { embed } from "ai";
import { PgVector } from "@mastra/pg";

// 質問をベクトルに変換
const { embedding } = await embed({
  value: "最も効果的な学習方法は？",
  model: openai.embedding('text-embedding-3-small'),
});

// データベースから関連情報を検索
const pgVector = new PgVector(process.env.POSTGRES_CONNECTION_STRING);
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,  // 10個の関連情報を呼び出す
});
```

すると、このような関連情報が返ってきます：

```typescript
[
  {
    text: "アクティブラーニングが最も効果的な学習方法の一つです...",
    score: 0.89,  // 89%の関連性！
    metadata: { source: "learning-methods.txt" }
  },
  {
    text: "反復学習と間隔を空けた復習は記憶定着に効果的です...",
    score: 0.85,
    metadata: { source: "memory-techniques.txt" }
  }
]
```

すごいですね！質問に関連する情報がすぐに見つかりました。まるで優秀な図書館員が「この質問に関する資料はこちらです」と素早く関連書籍を持ってきてくれるようなものです。

### 高度な検索テクニック 📊

単純な検索だけでなく、より複雑な条件でフィルタリングすることもできます：

```typescript
// メタデータを使ったフィルタリング付き検索
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  filter: {
    // カテゴリが「学習法」の情報だけを検索
    category: "学習法",
    // 2023年以降に作成された情報に限定
    createdAt: { $gte: "2023-01-01T00:00:00Z" },
  },
  topK: 5,
});
```

これで「学習法」カテゴリの最新情報だけが取得できます。古い情報や関係ないカテゴリの情報はスキップされるため、より精度の高い結果が得られます！

### ハイブリッド検索 - さらに精度を高める 🔍

ベクトル検索とキーワード検索を組み合わせた「ハイブリッド検索」も可能です。これにより、「意味的に近いけど、特定のキーワードが含まれていない」という情報も見つけられます：

```typescript
// ハイブリッド検索の例
const results = await vectorStore.hybridSearch({
  indexName: "embeddings",
  query: "効果的な学習方法",  // キーワード検索部分
  queryVector: embedding,    // 意味検索部分
  alpha: 0.7,                // ベクトル検索の重み（0-1）
  topK: 10,
});
```

このように、単語の一致だけでなく、意味的な関連性も考慮した検索ができるようになります。例えば「効果的な学習方法」で検索すると、「効率的な勉強テクニック」という似た意味の情報も見つけられるのです！

## RAGとエージェントの組み合わせ 🤖

RAGをエージェントに統合することで、「知識を持ったエージェント」が作れます。これが実践的なRAGエージェントの例です：

```typescript
import { Agent } from "@mastra/core/agent";
import { openai } from "@ai-sdk/openai";
import { createRAGTool } from "@mastra/rag/tools";
import { PgVector } from "@mastra/pg";

// ベクトルストアの設定
const vectorStore = new PgVector(process.env.POSTGRES_CONNECTION_STRING);

// RAGツールの作成
const ragTool = createRAGTool({
  id: "knowledgeBase",
  description: "学習方法に関する質問に答えるための知識ベース",
  vectorStore,
  indexName: "learning-methods",
  model: openai.embedding('text-embedding-3-small'),
  topK: 5,
});

// エージェントの定義
const learningAdvisorAgent = new Agent({
  name: "Learning Advisor",
  instructions: `あなたは学習アドバイザーです。ユーザーの学習に関する質問に、知識ベースの情報を活用して回答してください。
  知識ベースに情報がない場合は、その旨を正直に伝え、一般的なアドバイスを提供してください。`,
  model: openai("gpt-4o"),
  tools: {
    ragTool,
  },
});

// エージェントの使用例
const response = await learningAdvisorAgent.generate(
  "プログラミングを効率よく学ぶための方法は？"
);
console.log(response.text);
```

このエージェントに質問すると、次のような流れで処理されます：

1. エージェントが質問を受け取る（「プログラミングを効率よく学ぶ方法は？」）
2. ragToolがこの質問をベクトル化し、データベースで関連情報を検索
3. 検索結果が「コンテキスト」としてエージェントに提供される
4. エージェントはこのコンテキストを参考に、より正確な回答を生成

この仕組みにより、エージェントは自分の事前学習データだけでなく、あなたの用意した特定の知識ベースも活用できるようになります。結果として、より正確で、最新で、そして具体的な回答ができるようになるのです！

## RAGチューニングのベストプラクティス 🛠️

RAGシステムをより効果的にするためのいくつかのテクニックを紹介します：

### 1. チャンクサイズの最適化 📏

チャンクの大きさは重要です。大きすぎると不要な情報も含まれますが、小さすぎると文脈が失われます。通常、300〜500トークン程度が良いバランスですが、文書の種類によって調整する必要があります：

```typescript
// 文書タイプに応じたチャンク戦略
if (documentType === 'technicalManual') {
  // 技術マニュアルには小さめのチャンク
  return {
    strategy: 'recursive',
    size: 300,
    overlap: 30,
  };
} else if (documentType === 'narrative') {
  // 物語には大きめのチャンク（文脈を保持）
  return {
    strategy: 'recursive',
    size: 600,
    overlap: 100,
  };
}
```

### 2. メタデータと構造の活用 📋

メタデータを賢く使えば、検索精度が向上します：

```typescript
// メタデータを強化する例
const enhancedMetadata = {
  ...basicMetadata,
  // 章や節の情報を追加
  chapter: extractChapter(text),
  section: extractSection(text),
  // 読みやすさレベルを追加
  readabilityScore: calculateReadability(text),
  // 自動カテゴリ分類
  autoCategory: await categorizeText(text),
};
```

### 3. 検索結果の再ランキング 🏆

検索結果を取得した後、さらに最適な順序に並べ替えることができます：

```typescript
// 単純な検索
const initialResults = await vectorStore.query({
  indexName: "my-docs",
  queryVector: embedding,
  topK: 15, // より多くの候補を取得
});

// 追加のフィルタリングと再ランキング
const rerankedResults = initialResults
  // 関連性スコアが低すぎるものを除外
  .filter(result => result.score > 0.75)
  // 日付が新しいものを優先
  .sort((a, b) => {
    // まず関連性で並べ替え
    const relevanceDiff = b.score - a.score;
    if (Math.abs(relevanceDiff) > 0.1) return relevanceDiff;
    
    // 関連性が近い場合は日付で並べ替え
    return new Date(b.metadata.createdAt) - new Date(a.metadata.createdAt);
  })
  // 上位5つだけを使用
  .slice(0, 5);
```

### 4. ユーザーフィードバックの活用 👍

ユーザーからのフィードバックをRAGシステムの改善に活用することも重要です：

```typescript
// フィードバックを記録する関数
async function recordFeedback(query, results, feedback) {
  await db.insertFeedback({
    query,
    results: results.map(r => r.id),
    rating: feedback.rating,
    comment: feedback.comment,
    timestamp: new Date(),
  });
  
  // 特に低評価のケースを分析用にログ
  if (feedback.rating < 3) {
    console.log(`低評価クエリ: ${query}`);
    console.log(`返された結果: ${results.map(r => r.text).join('\n')}`);
    console.log(`ユーザーコメント: ${feedback.comment}`);
  }
}
```

こうしたフィードバックデータを分析することで、どのような質問でRAGが上手く機能していないかを特定し、システムを改善できます。

## 実践的RAGアプリケーション 🚀

最後に、Mastraを使った実践的なRAGアプリケーションの例を見てみましょう。これは社内ナレッジベースを作成し、従業員がチャットで質問できるようにするシステムです：

```typescript
import { Agent } from "@mastra/core/agent";
import { createRAGTool } from "@mastra/rag/tools";
import { createRouter } from "@ai-sdk/http";
import { openai } from "@ai-sdk/openai";
import { MDocument } from "@mastra/rag";
import { PgVector } from "@mastra/pg";

// 1. ベクトルストアの設定
const vectorStore = new PgVector(process.env.DATABASE_URL);

// 2. ドキュメント取り込み機能（管理者用）
async function ingestDocument(fileUrl, metadata) {
  // ファイルタイプに基づいてドキュメントを読み込む
  let doc;
  if (fileUrl.endsWith('.pdf')) {
    const pdfContent = await fetchPdfContent(fileUrl);
    doc = MDocument.fromText(pdfContent);
  } else if (fileUrl.endsWith('.md')) {
    const mdContent = await fetchContent(fileUrl);
    doc = MDocument.fromMarkdown(mdContent);
  } else {
    const textContent = await fetchContent(fileUrl);
    doc = MDocument.fromText(textContent);
  }
  
  // メタデータを追加
  doc.metadata = {
    ...metadata,
    source: fileUrl,
    ingestedAt: new Date().toISOString(),
  };
  
  // チャンク分割
  const chunks = await doc.chunk({
    strategy: "recursive",
    size: 400,
    overlap: 50,
  });
  
  // ベクトル化して保存
  const { embeddings } = await embedMany({
    model: openai.embedding("text-embedding-3-small"),
    values: chunks.map(chunk => chunk.text),
  });
  
  await vectorStore.upsert({
    indexName: "company-knowledge",
    vectors: embeddings,
    metadata: chunks.map((chunk, i) => ({
      ...doc.metadata,
      text: chunk.text,
      chunkId: `${doc.metadata.id}-chunk-${i}`,
    })),
  });
  
  return {
    status: "success",
    documentId: doc.metadata.id,
    chunkCount: chunks.length,
  };
}

// 3. RAGツールの作成
const companyKnowledgeTool = createRAGTool({
  id: "companyKnowledge",
  description: "会社の内部ドキュメント、ポリシー、手順に関する情報を提供します",
  vectorStore,
  indexName: "company-knowledge",
  model: openai.embedding("text-embedding-3-small"),
  topK: 5,
  // メタデータフィルタリング機能を追加
  metadataFilterBuilder: (query) => {
    // クエリから部門名を抽出（簡易実装）
    const departments = ["HR", "IT", "Finance", "Sales"];
    const deptMatch = departments.find(dept => 
      query.toLowerCase().includes(dept.toLowerCase())
    );
    
    // 部門名が見つかった場合はフィルタを追加
    return deptMatch 
      ? { department: deptMatch } 
      : {};
  }
});

// 4. エージェントの定義
const knowledgeAgent = new Agent({
  name: "Company Knowledge Assistant",
  instructions: `あなたは会社の知識アシスタントです。
会社のポリシー、手順、ガイドラインに関する質問に対して、
提供された知識ベースの情報を使って正確に回答してください。

以下の原則を守ってください：
1. 知識ベースに情報がある場合は、その情報に基づいて回答
2. 情報がない場合は、その旨を率直に伝える
3. 推測や不確かな情報は提供しない
4. 機密情報や個人情報に関わる質問には適切に対応する
5. 回答の根拠となる情報源（ドキュメント名）を示す

知識ベースの情報更新日：${new Date().toISOString().split('T')[0]}`,
  model: openai("gpt-4o"),
  tools: {
    companyKnowledgeTool,
  },
});

// 5. APIエンドポイントの設定
const router = createRouter({
  agent: knowledgeAgent,
});

// エクスポート
export { router as knowledgeRouter, ingestDocument };
```

このコードは、会社の内部ドキュメントを取り込み、従業員が質問できるチャットボットを作成するものです。特徴は：

1. **多様な文書形式対応**: PDF、マークダウン、テキストなど様々な形式の文書を取り込める
2. **メタデータフィルタリング**: 「HR部門のポリシーについて」といった質問では、自動的にHR関連文書だけを検索
3. **情報の鮮度表示**: 知識ベースの更新日をユーザーに表示
4. **APIエンドポイント**: フロントエンドアプリケーションから簡単に利用できる

このようなシステムがあれば、新入社員のオンボーディングや、社内ポリシーの問い合わせ対応が格段に効率化されます。しかも、最新の情報に基づいた回答が得られるため、情報の正確性も向上するでしょう！

## まとめ - RAGの可能性 🌟

この章では、RAG（検索拡張生成）の基本から応用まで、幅広く学んできました：

- RAGの基本概念と重要性
- 文書の収集と前処理の方法
- ベクトル埋め込みの仕組み
- ベクトルデータベースの選択と使用法
- 検索とフィルタリングの技術
- エージェントとRAGの統合方法
- 実践的なRAGアプリケーションの構築

RAGは単なる技術ではなく、AIの新たな可能性を開く鍵です。これにより、あなたのエージェントは：

- より正確で具体的な回答ができる
- 最新の情報に基づいた助言ができる
- 特定のドメイン知識を持った専門家になれる
- ハルシネーション（幻覚・誤情報）を大幅に減らせる

次の章では、Mastraのローカル開発環境について学び、実際にこれらの技術を試す方法を見ていきましょう。あなただけの知識豊かなAIエージェントを構築する冒険は、まだ始まったばかりです！ 