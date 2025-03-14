---
title: "RAG（検索拡張生成）"
--- 

# RAG（検索拡張生成）

## RAGの概要と基本原理

RAG（Retrieval-Augmented Generation：検索拡張生成）は、大規模言語モデル（LLM）の出力を、あなた独自のデータソースから取得した関連情報で強化する技術です。MastraのRAGシステムは、以下の機能を提供します：

- ドキュメントの処理と埋め込みのための標準化されたAPI
- 複数のベクトルストアのサポート
- 最適な検索のためのチャンキングと埋め込み戦略
- 埋め込みと検索のパフォーマンスを追跡するための観測機能

RAGを実装することで、エージェントの応答がより正確になり、最新の情報に基づいた回答ができるようになります。エージェントは自分の知識に加えて、特定の文書や専門的な情報を参照できるため、より確かな根拠に基づいた応答が可能になります。

## RAGの基本的な流れ

RAGの基本的な流れは次のとおりです：

1. **ドキュメント処理**: テキスト、HTML、マークダウンなどの形式からドキュメントを読み込み
2. **チャンキング**: ドキュメントを管理しやすい小さな断片（チャンク）に分割
3. **埋め込み生成**: チャンクをベクトル表現（埋め込み）に変換
4. **ベクトルストア保存**: 埋め込みをベクトルデータベースに保存
5. **検索**: クエリに対して、最も関連性の高いチャンクを検索
6. **再ランキング**: 必要に応じて検索結果を精緻化
7. **応答生成**: 検索結果を使用してLLMが応答を生成

以下は基本的なRAG実装の例です：

```typescript
import { embedMany } from "ai";
import { openai } from "@ai-sdk/openai";
import { PgVector } from "@mastra/pg";
import { MDocument } from "@mastra/rag";

// 1. ドキュメントの初期化
const doc = MDocument.fromText(`ここにドキュメントのテキストを入力...`);

// 2. チャンクの作成
const chunks = await doc.chunk({
  strategy: "recursive",
  size: 512,
  overlap: 50,
});

// 3. 埋め込みの生成
const { embeddings } = await embedMany({
  values: chunks,
  model: openai.embedding("text-embedding-3-small"),
});

// 4. ベクトルデータベースに保存
const pgVector = new PgVector(process.env.POSTGRES_CONNECTION_STRING);
await pgVector.upsert({
  indexName: "embeddings",
  vectors: embeddings,
});

// 5. 類似チャンクのクエリ
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: queryVector, // クエリの埋め込み
  topK: 3,
});

console.log("類似チャンク:", results);
```

## ドキュメント処理とチャンキング

RAGの基本的な構成要素はドキュメント処理です。まず、`MDocument`インスタンスを作成して、様々な形式からドキュメントを初期化できます：

```typescript
const docFromText = MDocument.fromText("プレーンテキストの内容...");
const docFromHTML = MDocument.fromHTML("<html>HTMLの内容...</html>");
const docFromMarkdown = MDocument.fromMarkdown("# マークダウンの内容...");
const docFromJSON = MDocument.fromJSON(`{ "key": "value" }`);
```

### チャンキング戦略

ドキュメントを管理可能な断片に分割するには、`chunk`メソッドを使用します。Mastraは異なるドキュメントタイプに最適化された複数のチャンキング戦略をサポートしています：

* `recursive`: コンテンツ構造に基づくスマートな分割
* `character`: シンプルな文字ベースの分割
* `token`: トークンを考慮した分割
* `markdown`: マークダウンを考慮した分割
* `html`: HTML構造を考慮した分割
* `json`: JSON構造を考慮した分割
* `latex`: LaTeX構造を考慮した分割

以下は`recursive`戦略を使用する例です：

```typescript
const chunks = await doc.chunk({
  strategy: "recursive",
  size: 512,        // チャンクのサイズ
  overlap: 50,      // オーバーラップするトークン数
  separator: "\n",  // 分割の区切り文字
  extract: {
    metadata: true, // メタデータ抽出（オプション）
  },
});
```

## 埋め込み生成

チャンクをベクトル表現に変換するには、OpenAIやCohereなどの埋め込みプロバイダーを使用します：

### OpenAIを使用する場合

```typescript
import { openai } from "@ai-sdk/openai";
import { embedMany } from "ai";

const { embeddings } = await embedMany({
  model: openai.embedding('text-embedding-3-small'),
  values: chunks.map(chunk => chunk.text),
});
```

### Cohereを使用する場合

```typescript
import { embedMany } from 'ai';
import { cohere } from '@ai-sdk/cohere';

const { embeddings } = await embedMany({
  model: cohere.embedding('embed-english-v3.0'),
  values: chunks.map(chunk => chunk.text),
});
```

埋め込み関数は、テキストの意味的な表現であるベクトル（数値の配列）を返します。これらのベクトルは、ベクトルデータベースでの類似度検索に使用されます。

## ベクトルデータベース

埋め込みを生成した後、それらをベクトル類似度検索をサポートするデータベースに保存する必要があります。Mastraは様々なベクトルデータベースを統一的なインターフェースでサポートしています。

### サポートされているデータベース

Mastraは以下のベクトルデータベースをサポートしています：

- **PostgreSQL with pgvector**: すでにPostgreSQLを使用しているチームに最適
- **Pinecone**: スケーラブルなベクトル検索に特化
- **Qdrant**: 高パフォーマンスなベクトル検索
- **Chroma**: オープンソースのベクトルデータベース
- **Astra DB**: DataStaxが提供するクラウドネイティブデータベース
- **LibSQL**: SQLiteベースのベクトルデータベース
- **Upstash**: サーバーレスデータベース
- **Cloudflare Vectorize**: Cloudflareのエッジベクトルデータベース

例として、PostgreSQLとpgvectorを使用する場合：

```typescript
import { PgVector } from '@mastra/pg';

const store = new PgVector(process.env.POSTGRES_CONNECTION_STRING);

// インデックスの作成
await store.createIndex({
  indexName: "my-collection",
  dimension: 1536,  // text-embedding-3-smallの次元数
});

// 埋め込みの保存
await store.upsert({
  indexName: "my-collection",
  vectors: embeddings,
  metadata: chunks.map(chunk => ({
    text: chunk.text,
    // メタデータを追加
    source: chunk.source,
    category: chunk.category,
    createdAt: new Date().toISOString(),
  })),
});
```

### メタデータの追加

ベクトルストアには、高度なフィルタリングと整理のためのリッチなメタデータをサポートしています。検索時に使用したいJSONシリアライズ可能なフィールドを追加できます：

```typescript
await vectorStore.upsert({
  indexName: "embeddings",
  vectors: embeddings,
  metadata: chunks.map((chunk) => ({
    // 基本的な内容
    text: chunk.text,
    id: chunk.id,
    // ドキュメント整理
    source: chunk.source,
    category: chunk.category,
    // 時間関連のメタデータ
    createdAt: new Date().toISOString(),
    version: "1.0",
    // カスタムフィールド
    language: chunk.language,
    author: chunk.author,
  })),
});
```

## 検索と取得

ベクトルストアに埋め込みを保存した後、ユーザークエリに応答するために関連するチャンクを取得する必要があります。Mastraは、意味検索、フィルタリング、再ランキングをサポートする柔軟な検索オプションを提供します。

### 基本的な検索

最も簡単なアプローチは直接的な意味検索です。この方法では、ベクトル類似度を使用してクエリと意味的に類似したチャンクを見つけます：

```typescript
import { openai } from "@ai-sdk/openai";
import { embed } from "ai";
import { PgVector } from "@mastra/pg";

// クエリを埋め込みに変換
const { embedding } = await embed({
  value: "記事の要点は何ですか？",
  model: openai.embedding('text-embedding-3-small'),
});

// ベクトルストアのクエリ
const pgVector = new PgVector(process.env.POSTGRES_CONNECTION_STRING);
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,
});
```

結果にはテキスト内容と類似度スコアが含まれます：

```typescript
[
  {
    text: "気候変動は重要な課題を提起しています...",
    score: 0.89,
    metadata: { source: "article1.txt" }
  },
  {
    text: "気温上昇は作物収量に影響します...",
    score: 0.82,
    metadata: { source: "article1.txt" }
  }
  // ... その他の結果
]
```

### メタデータフィルタリング

メタデータフィールドに基づいて結果をフィルタリングして、検索範囲を絞り込むことができます。これは、異なるソース、時間帯、または特定の属性を持つドキュメントがある場合に役立ちます：

```typescript
// シンプルな等価フィルター
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,
  filter: { source: "article1.txt" }
});

// 数値比較
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,
  filter: { price: { $gt: 100 } }
});

// 複数の条件
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,
  filter: {
    category: "electronics",
    price: { $lt: 1000 },
    inStock: true
  }
});
```

### 再ランキング

初期のベクトル類似度検索では、微妙な関連性を見逃すことがあります。再ランキングは、より計算コストが高いですが、より正確なアルゴリズムで結果を改善します：

```typescript
import { openai } from "@ai-sdk/openai";
import { rerank } from "@mastra/rag";

// ベクトル検索から初期結果を取得
const initialResults = await pgVector.query({
  indexName: "embeddings",
  queryVector: queryEmbedding,
  topK: 10,
});

// 結果を再ランキング
const rerankedResults = await rerank(initialResults, query, openai('gpt-4o-mini'));
```

再ランキングされた結果は、ベクトル類似度と意味的な理解を組み合わせて検索品質を向上させます。

## RAGエージェントの作成

Mastraでは、ベクトルクエリツールを使用してRAGエージェントを簡単に作成できます：

```typescript
import { openai } from "@ai-sdk/openai";
import { createVectorQueryTool } from "@mastra/rag";
import { Agent } from "@mastra/core";
import { PgVector } from "@mastra/pg";
import { PGVECTOR_PROMPT } from "@mastra/rag";

// ベクトルストアの初期化
const pgVector = new PgVector(process.env.POSTGRES_CONNECTION_STRING);

// ベクトルクエリツールの作成
const vectorQueryTool = createVectorQueryTool({
  vectorStoreName: 'pgVector',
  indexName: 'embeddings',
  model: openai.embedding('text-embedding-3-small'),
});

// RAGエージェントの定義
export const ragAgent = new Agent({
  name: 'RAG Agent',
  model: openai('gpt-4o-mini'),
  instructions: `
    提供されたコンテキストを使用してクエリを処理してください。
    レスポンスは簡潔で関連性のある構造にしてください。
    ${PGVECTOR_PROMPT}
  `,
  tools: { vectorQueryTool },
});
```

## 高度なRAG実装：Chain of Thought

より複雑なクエリに対応するために、Chain of Thought（思考の連鎖）ワークフローを使用した高度なRAG実装も可能です：

```typescript
import { Workflow } from "@mastra/core";
import { openai } from "@ai-sdk/openai";

const cotRagWorkflow = new Workflow({
  name: "Chain of Thought RAG",
});

// ステップ1: クエリを分析し、検索戦略を計画
cotRagWorkflow.step({
  name: "query-analysis",
  execute: async ({ input }) => {
    const analysis = await openai("gpt-4o-mini").generate({
      messages: [
        { role: "system", content: "クエリを分析し、最適な検索戦略を提案してください。" },
        { role: "user", content: input.query }
      ]
    });
    return { analysis: analysis.content, query: input.query };
  }
});

// ステップ2: 検索の実行
cotRagWorkflow.step({
  name: "retrieval",
  after: ["query-analysis"],
  execute: async ({ input, tools }) => {
    const results = await tools.vectorQueryTool(input.query);
    return { ...input, results };
  }
});

// ステップ3: 結果の評価と不足情報の特定
cotRagWorkflow.step({
  name: "evaluation",
  after: ["retrieval"],
  execute: async ({ input }) => {
    const evaluation = await openai("gpt-4o-mini").generate({
      messages: [
        { role: "system", content: "検索結果を評価し、不足している情報を特定してください。" },
        { role: "user", content: `クエリ: ${input.query}\n\n検索結果: ${JSON.stringify(input.results)}` }
      ]
    });
    return { ...input, evaluation: evaluation.content };
  }
});

// ステップ4: 最終回答の生成
cotRagWorkflow.step({
  name: "answer-generation",
  after: ["evaluation"],
  execute: async ({ input }) => {
    const answer = await openai("gpt-4o-mini").generate({
      messages: [
        { role: "system", content: "検索結果と評価に基づいて包括的な回答を生成してください。" },
        { role: "user", content: `クエリ: ${input.query}\n\n検索結果: ${JSON.stringify(input.results)}\n\n評価: ${input.evaluation}` }
      ]
    });
    return { answer: answer.content };
  }
});
```

## RAGのベストプラクティス

RAGを効果的に実装するためのいくつかのベストプラクティスを紹介します：

1. **適切なチャンクサイズの選択**:
   - 小さすぎるチャンクは文脈を失います
   - 大きすぎるチャンクは精度を下げます
   - コンテンツタイプに合わせてサイズを調整（通常は256〜1024トークン）

2. **メタデータを活用した検索フィルタリング**:
   - ドキュメントソース、作成日、カテゴリなどでフィルタリング
   - 検索範囲を絞り込んで精度を向上

3. **再ランキングによる結果の改善**:
   - ベクトル検索の結果を再ランキングして精度を向上
   - より多くの候補を検索し、上位の結果を再ランキング

4. **プロンプトエンジニアリングの活用**:
   - 検索結果をLLMに効果的に伝えるプロンプトを設計
   - 引用や参照を適切に示すよう指示

5. **RAGパイプラインの評価と改善**:
   - 検索精度、応答の正確さ、ハルシネーション（幻覚）の発生率を評価
   - 継続的にシステムを調整

## まとめ

RAG（検索拡張生成）は、大規模言語モデルの能力を外部のデータソースと組み合わせることで、より正確で関連性の高い応答を生成する強力な技術です。Mastraでは、ドキュメント処理からチャンキング、埋め込み生成、ベクトルストレージ、そして高度な検索機能まで、RAGパイプライン全体を実装するための包括的なツールセットを提供しています。

効果的なRAG実装により、エージェントは最新の情報に基づいた回答ができるようになり、ハルシネーションを減らし、より確かな根拠に基づいた応答が可能になります。本章で紹介した技術とベストプラクティスを活用して、独自のRAGソリューションを構築してみてください。 