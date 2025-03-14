---
title: "RAG（検索拡張生成）"
--- 

# RAGの魔法を解き放つ 🔮✨

## RAGとは？魔法使いの記憶術 📚

RAG（Retrieval-Augmented Generation：検索拡張生成）とは、魔法使い（LLM）の知識を、あなた独自の秘密の書物（データソース）から取り出した情報で強化する特別な魔術です！これは、魔法使いが自分自身の記憶だけでなく、あなたが用意した魔導書も参照して、より正確な呪文を唱えられるようにする技術なんです。✨

Mastraの魔法図書館（RAGシステム）は、以下の素晴らしい機能を提供します：

- 📜 魔法の書物（ドキュメント）を処理し、記憶の結晶（埋め込み）に変える標準化された魔法の杖（API）
- 🗄️ 様々な魔法の記憶庫（ベクトルストア）と連携できる
- ✂️ 書物を最適な断片（チャンク）に分割し、記憶しやすくする戦略
- 📊 魔法の効果（埋め込みと検索のパフォーマンス）を見える化する水晶球

「でも難しそう...」と思いましたか？心配いりません！RAGの魔法を習得すれば、あなたのエージェントはより賢く、正確になり、最新の情報を使った回答ができるようになります。まるで、魔法使いに最新の百科事典や専門書を与えるようなものです。結果として、「ふーん、そう思うだけかい？」という曖昧な回答から、「実はこれには確かな根拠があります！」という信頼性の高い回答へと進化するのです！🌟

## RAGの魔法の詠唱手順 🪄

RAGの魔法は、次のような美しい流れで詠唱されます：

1. **書物の収集** 📚: テキスト、HTML、マークダウンなどの形式から魔法の知識を集める
2. **知識の分割** ✂️: 大きな書物を理解しやすい小さな断片（チャンク）に分ける
3. **記憶の結晶化** 💎: 断片を魔法のエネルギー（ベクトル）に変換する
4. **記憶の保管** 🗃️: エネルギーを魔法の保管庫（ベクトルデータベース）に収める
5. **知識の検索** 🔍: 質問に関連する最も価値ある記憶の断片を呼び出す
6. **記憶の整理** 📋: 必要に応じて記憶を重要度順に並べ直す
7. **魔法の回答** ✨: 集めた記憶を使って魔法使い（LLM）が最適な回答を紡ぎ出す

それでは、基本的なRAG魔法の呪文を見てみましょう：

```typescript
import { embedMany } from "ai";
import { openai } from "@ai-sdk/openai";
import { PgVector } from "@mastra/pg";
import { MDocument } from "@mastra/rag";

// 1. 魔法の書物を用意
const doc = MDocument.fromText(`ここに秘密の知識を記入...`);

// 2. 書物を読みやすい断片に分割
const chunks = await doc.chunk({
  strategy: "recursive",
  size: 512,
  overlap: 50,
});

// 3. 断片を魔法のエネルギーに変換
const { embeddings } = await embedMany({
  values: chunks,
  model: openai.embedding("text-embedding-3-small"),
});

// 4. エネルギーを魔法の保管庫に収める
const pgVector = new PgVector(process.env.POSTGRES_CONNECTION_STRING);
await pgVector.upsert({
  indexName: "embeddings",
  vectors: embeddings,
});

// 5. 関連する知識を検索
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: queryVector, // 質問のエネルギー
  topK: 3,
});

console.log("見つけた関連知識:", results);
```

「うわぁ、難しそう...」と思ったかもしれませんね。でも大丈夫！これは単に「本を集めて→分けて→記憶して→必要な時に思い出す」という、私たちが日常的に行っていることを、コンピュータ言語で表現しただけなんです。一歩ずつ進めていきましょう！🚶‍♀️

## 魔法の書物を整える 📚✨

RAGの第一歩は、魔法の書物（ドキュメント）を集めることです。Mastraでは、`MDocument`という魔法の道具を使って、さまざまな形式の知識を取り込むことができます：

```typescript
const docFromText = MDocument.fromText("昔々あるところに...");
const docFromHTML = MDocument.fromHTML("<html>魔法の王国の歴史...</html>");
const docFromMarkdown = MDocument.fromMarkdown("# 魔法の基本原則...");
const docFromJSON = MDocument.fromJSON(`{ "魔法": "火の呪文" }`);
```

これはまるで、図書館の司書が様々な形式の本（テキスト、ウェブページ、マークダウン文書、データ集）を集めているようなものですね！📚

### 知識を小さく分ける術 ✂️

大きな書物をそのまま記憶するのは大変です。そこで、`chunk`という魔法で書物を小さな断片に分けます。Mastraは様々な分け方を知っていて、書物の種類に合わせた最適な分割方法を選べます：

* `recursive` 🌲: 内容の構造を考慮した賢い分割（まるで目次から章、節へと分けるよう）
* `character` 📏: シンプルに文字数で区切る方法
* `token` 🔤: 言葉の意味の単位で区切る方法
* `markdown` 📝: マークダウン形式の文書に適した分割
* `html` 🌐: ウェブページの構造に合わせた分割
* `json` 📊: データ構造に合わせた分割
* `latex` 📑: 学術論文などの特殊な形式に合わせた分割

例えば、賢い分割方法（recursive）を使うにはこんな魔法の呪文を唱えます：

```typescript
const chunks = await doc.chunk({
  strategy: "recursive",  // 賢い分割方法を選択
  size: 512,              // だいたいこのくらいの大きさに
  overlap: 50,            // 少し重複させて、文脈を失わないように
  separator: "\n",        // 段落ごとに区切るのがいいかも
  extract: {
    metadata: true,       // 書物の情報（著者、日付など）も取っておく
  },
});
```

こうして、大きな魔法書が読みやすい小さなページに分かれました。これで記憶するのも、必要な部分を探すのも簡単になります！🧠✨

## 魔法のエネルギーへの変換 💫

さて、集めた知識の断片を魔法のエネルギー（ベクトル埋め込み）に変換しましょう。これは言葉の意味を数値の配列に変換する、まさに「言葉を魔法のエネルギーに変える」神秘的な過程です！

### OpenAIの魔法で変換する場合 🔮

```typescript
import { openai } from "@ai-sdk/openai";
import { embedMany } from "ai";

const { embeddings } = await embedMany({
  model: openai.embedding('text-embedding-3-small'),
  values: chunks.map(chunk => chunk.text),
});
```

### Cohereの魔法で変換する場合 🌈

```typescript
import { embedMany } from 'ai';
import { cohere } from '@ai-sdk/cohere';

const { embeddings } = await embedMany({
  model: cohere.embedding('embed-english-v3.0'),
  values: chunks.map(chunk => chunk.text),
});
```

この魔法が生み出すのは、テキストの意味を表す数字の配列。人間には単なる数字の羅列に見えますが、コンピューターにとっては「ああ、これは猫についての文章で、これは天気についての文章だ」と意味を理解できる、魔法のエネルギーなのです。✨

## 魔法の記憶庫 🏰

エネルギーに変換した知識は、特別な魔法の記憶庫（ベクトルデータベース）に保管します。Mastraは様々な記憶庫と連携できるので、あなたの冒険に最適なものを選べます！

### 選べる魔法の記憶庫 🗄️

- **PostgreSQL with pgvector** 🐘: すでにPostgreSQLという魔法を使っている冒険者に最適
- **Pinecone** 🌲: 大量の記憶を高速に検索したい上級魔法使いに
- **Qdrant** ⚡: パワフルな魔法検索が必要な時に
- **Chroma** 🌈: 誰でも使える魔法の記憶庫
- **Astra DB** 🌠: 雲の上の魔法図書館
- **LibSQL** 📱: 小さな装置でも使える軽量な魔法記憶庫
- **Upstash** ☁️: サーバーレスな魔法の世界で
- **Cloudflare Vectorize** 🌐: 世界中どこでも高速アクセスできる魔法の記憶庫

例えば、PostgreSQLという魔法の記憶庫を使うなら、こんな魔法の呪文を唱えます：

```typescript
import { PgVector } from '@mastra/pg';

const store = new PgVector(process.env.POSTGRES_CONNECTION_STRING);

// 記憶庫の棚を準備
await store.createIndex({
  indexName: "my-collection",
  dimension: 1536,  // 魔法のエネルギーの大きさ
});

// 記憶を収納
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

### 記憶の整理術 📋

魔法の記憶には、「どの本から来たか」「いつの情報か」といった情報（メタデータ）をつけておくと、後で探しやすくなります。これは図書館で本に「分類ラベル」や「出版日」を記録するようなものです：

```typescript
await vectorStore.upsert({
  indexName: "embeddings",
  vectors: embeddings,
  metadata: chunks.map((chunk) => ({
    // 基本情報
    text: chunk.text,
    id: chunk.id,
    // 書物の整理情報
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

これで魔法の記憶庫に、きちんと整理された知識が保管されました。「魔法の火属性の呪文だけ」「去年書かれた情報だけ」などと指定して検索できるようになります！📚✨

## 魔法の知識を呼び出す 🔍

さあ、保管した魔法の知識を実際に使ってみましょう！質問が来たら、関連する記憶を呼び出して、最適な回答を作る冒険の始まりです。

### 基本的な記憶検索 🧙‍♂️

最もシンプルな方法は、質問を同じく魔法のエネルギーに変換して、似たエネルギーを持つ記憶を探す方法です：

```typescript
import { openai } from "@ai-sdk/openai";
import { embed } from "ai";
import { PgVector } from "@mastra/pg";

// 質問を魔法のエネルギーに変換
const { embedding } = await embed({
  value: "ドラゴンを倒す最善の方法は？",
  model: openai.embedding('text-embedding-3-small'),
});

// 魔法の記憶庫から関連知識を検索
const pgVector = new PgVector(process.env.POSTGRES_CONNECTION_STRING);
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,  // 10個の関連記憶を呼び出す
});
```

すると、このような関連知識が返ってきます：

```typescript
[
  {
    text: "ドラゴンは鱗の下が弱点です...",
    score: 0.89,  // 89%の関連性！
    metadata: { source: "dragon-manual.txt" }
  },
  {
    text: "炎のブレスに対抗するには魔法の盾が効果的...",
    score: 0.82,
    metadata: { source: "dragon-manual.txt" }
  }
  // ... その他の知識
]
```

これで質問に最も関連する魔法の知識が手に入りました！まるで「ドラゴン退治の専門家」に質問しているようですね！🐉

### 記憶の選別術 🧠

時には、特定の本や分野からだけ知識を引き出したいこともあるでしょう。そんな時はメタデータフィルタリングという魔法で、記憶を選別できます：

```typescript
// 特定の書物からだけ知識を取り出す
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,
  filter: { source: "dragon-manual.txt" }  // ドラゴン専門の本だけから
});

// 価格が100ゴールド以上のアイテムについての知識だけ
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,
  filter: { price: { $gt: 100 } }
});

// より複雑な条件で絞り込む
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,
  filter: {
    category: "electronics",  // 電子機器についての
    price: { $lt: 1000 },     // 1000ゴールド未満の
    inStock: true             // 在庫がある商品だけ
  }
});
```

これで必要な記憶だけを精密に取り出せるようになりました！いらない情報に惑わされることなく、最適な答えを見つけられます。🎯

### 記憶の再評価 ⚖️

初めに見つけた記憶が、必ずしも最適とは限りません。そこで登場するのが「再ランキング」という高度な魔法。これは「まずざっと関連しそうな記憶を集めて、その後じっくり評価し直す」という二段階の魔法です：

```typescript
import { openai } from "@ai-sdk/openai";
import { rerank } from "@mastra/rag";

// まずは幅広く記憶を検索
const initialResults = await pgVector.query({
  indexName: "embeddings",
  queryVector: queryEmbedding,
  topK: 10,
});

// 集めた記憶を、より賢い魔法で再評価
const rerankedResults = await rerank(initialResults, query, openai('gpt-4o-mini'));
```

これにより、単純な類似度では見逃していた「実は最も関連が深い」知識が浮かび上がってきます。まるで「熟練の司書が本を厳選してくれる」ような、高度な魔法です！📚✨

## 魔法の知識を持つアシスタントの創造 🧙‍♀️

いよいよ、集めた魔法の知識を使いこなす「RAGエージェント」を作りましょう！これは、自分の記憶と外部の知識を組み合わせて答えを出す、賢い魔法使いのようなものです：

```typescript
import { openai } from "@ai-sdk/openai";
import { createVectorQueryTool } from "@mastra/rag";
import { Agent } from "@mastra/core";
import { PgVector } from "@mastra/pg";
import { PGVECTOR_PROMPT } from "@mastra/rag";

// 魔法の記憶庫を準備
const pgVector = new PgVector(process.env.POSTGRES_CONNECTION_STRING);

// 記憶を検索する魔法の道具を作成
const vectorQueryTool = createVectorQueryTool({
  vectorStoreName: 'pgVector',
  indexName: 'embeddings',
  model: openai.embedding('text-embedding-3-small'),
});

// 魔法使いの創造
export const ragAgent = new Agent({
  name: '知識の魔法使い',
  model: openai('gpt-4o-mini'),
  instructions: `
    あなたは膨大な知識を持つ賢者です。
    提供された情報を元に、正確で役立つ回答を作ってください。
    わからないことは正直に「わかりません」と答えましょう。
    ${PGVECTOR_PROMPT}
  `,
  tools: { vectorQueryTool },
});
```

これで完成したRAGエージェントは、質問に対して自分の基本知識と、あなたが用意した特別な魔法の書物の知識を組み合わせて、より正確で信頼性の高い回答を作れるようになりました！🧠✨

## 上級魔法：思考の連鎖 🔄

より難しい質問に対応するために、「思考の連鎖（Chain of Thought）」という上級魔法もあります。これは「まず問題を分析し→計画を立て→情報を集め→評価して→最終的に答える」という、人間の思考プロセスを模倣した高度な魔法です：

```typescript
import { Workflow } from "@mastra/core";
import { openai } from "@ai-sdk/openai";

const cotRagWorkflow = new Workflow({
  name: "思考の連鎖",
});

// ステップ1: 問題を分析し、検索戦略を考える
cotRagWorkflow.step({
  name: "query-analysis",
  execute: async ({ input }) => {
    const analysis = await openai("gpt-4o-mini").generate({
      messages: [
        { role: "system", content: "質問を深く分析し、どんな情報が必要か考えてください" },
        { role: "user", content: input.query }
      ]
    });
    return { analysis: analysis.content, query: input.query };
  }
});

// ステップ2: 実際に情報を検索
cotRagWorkflow.step({
  name: "retrieval",
  after: ["query-analysis"],
  execute: async ({ input, tools }) => {
    const results = await tools.vectorQueryTool(input.query);
    return { ...input, results };
  }
});

// ステップ3: 集めた情報を評価し、足りない点を確認
cotRagWorkflow.step({
  name: "evaluation",
  after: ["retrieval"],
  execute: async ({ input }) => {
    const evaluation = await openai("gpt-4o-mini").generate({
      messages: [
        { role: "system", content: "集めた情報を評価し、不足している点を指摘してください" },
        { role: "user", content: `質問: ${input.query}\n\n検索結果: ${JSON.stringify(input.results)}` }
      ]
    });
    return { ...input, evaluation: evaluation.content };
  }
});

// ステップ4: 最終的な回答を作成
cotRagWorkflow.step({
  name: "answer-generation",
  after: ["evaluation"],
  execute: async ({ input }) => {
    const answer = await openai("gpt-4o-mini").generate({
      messages: [
        { role: "system", content: "すべての情報を考慮して、最高の回答を作成してください" },
        { role: "user", content: `質問: ${input.query}\n\n検索結果: ${JSON.stringify(input.results)}\n\n評価: ${input.evaluation}` }
      ]
    });
    return { answer: answer.content };
  }
});
```

これは複数の魔法使いが協力して問題を解決するような、高度な魔法のコラボレーションです。一人では思いつかない完璧な答えが生まれる秘訣ですね！🧙‍♂️🧙‍♀️

## 魔法を極めるための秘訣 💯

RAGの魔法をマスターするための、賢者たちの秘伝を伝授します：

1. **魔法の断片の大きさ** 📏:
   - 小さすぎると文脈が失われる
   - 大きすぎると精度が下がる
   - コンテンツの種類に合わせて調整（通常は256〜1024トークン）
   - 賢者の助言：「本の一章丸ごとではなく、数段落程度が最適」

2. **魔法の記憶の整理術** 🗂️:
   - 書物の出所、作成日、カテゴリなどの情報を活用
   - 検索範囲を絞って、より精度の高い回答を
   - 賢者の助言：「整理された図書館からは、最高の知識が見つかる」

3. **記憶の選別の技** 🔍:
   - より多くの候補を集めてから、最良のものを選ぶ
   - 賢者の助言：「100の本を軽く見てから、最適な10冊を精読する」

4. **魔法使いへの伝え方** 📢:
   - 集めた知識をLLMに効果的に伝える言葉選び
   - 引用や出典を明記するよう指示
   - 賢者の助言：「魔法使いにも、明確な指示が必要」

5. **魔法の効果測定** 📊:
   - 検索精度、回答の正確さ、幻覚（実際には存在しない情報）の発生率をチェック
   - 継続的に改善
   - 賢者の助言：「真の魔法使いは、自らの魔法を常に磨き続ける」

## まとめ：知識の魔法、習得完了！ 🎓✨

おめでとうございます！RAG（検索拡張生成）という強力な魔法の基礎を学び終えました。この魔法は、AIの魔法使い（LLM）の能力を、あなた独自の魔法書（データソース）と組み合わせることで、より正確で信頼できる応答を生み出す素晴らしい技術です。

Mastraの魔法キットを使えば、書物の収集から断片化、エネルギー変換、記憶の保管、そして高度な検索まで、RAGの魔法を隅々まで実装できます。この魔法を習得することで、あなたのエージェントは、単なる一般的な知識だけではなく、あなた独自の情報も活用して、より賢く、より役立つ助言ができるようになります。

「想像上の答え」から「確かな根拠に基づいた回答」へ。その変化は、まるで見習い魔法使いから賢者への成長のようです。

今こそ、この章で学んだ魔法の技術とコツを活用して、あなただけの魔法の図書館を構築し、究極の魔法のアシスタントを創造する冒険に出発しましょう！🧙‍♂️📚✨ 