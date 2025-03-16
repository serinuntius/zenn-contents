---
title: "第5章：RAGツールの実装（tools/rag）"
---

# RAGツールで知識を強化しよう！🧠✨

前章までに、GitHubからコードリポジトリをクローンし、その構造を詳細に分析するツールを実装しました。素晴らしい進歩です！でも、AIエージェントがコードを本当に「理解」するには、さらに一歩踏み込む必要があります。そこで登場するのが「RAG（Retrieval Augmented Generation）」、つまり「検索拡張生成」という強力な技術です！

この章では、RAGの核となる2つの重要なツールを実装していきます。まずは、ファイルをチャンキングして埋め込みベクトルに変換し、ベクトルデータベースに保存する「fileProcessorTool」から始めましょう。これがRAGの基盤となる部分です！🚀

## ファイル処理ツール (fileProcessorTool) 📄→💾

**ファイルパス**: `src/mastra/tools/rag/fileProcessor.ts`

fileProcessorToolは、コードファイルやドキュメントを取り込み、それを小さなチャンク（断片）に分割し、各チャンクをベクトル表現に変換してデータベースに保存します。これにより、AIエージェントは膨大なコードベースの中から関連情報を瞬時に検索できるようになります！

```typescript
import { createTool } from "@mastra/core/tools";
import { z } from "zod";
import fs from "fs/promises";
import path from "path";
import { MDocument } from "@mastra/rag";
import { embedMany } from "ai";
import { LibSQLVector } from "@mastra/core/vector/libsql";
import { googleEmbeddingModel } from "../../models";

/**
 * ファイルを処理してチャンキング、ベクトル化、保存を行うツール
 */
export const fileProcessorTool = createTool({
    id: "file-processor",
    description:
        "ファイルを受け取り、チャンキングしてベクトルストアに保存します",
    inputSchema: z.object({
        filePath: z.string().describe("処理するファイルのパス"),
        strategy: z
            .enum([
                "recursive",
                "character",
                "token",
                "markdown",
                "html",
                "json",
                "latex",
            ])
            .default("recursive")
            .describe("チャンキング戦略"),
        chunkSize: z.number().default(512).describe("チャンクのサイズ"),
        overlap: z.number().default(50).describe("チャンク間の重複トークン数"),
        separator: z.string().default("\n").describe("区切り文字"),
        dbPath: z
            .string()
            .default("vector_store.db")
            .describe("ベクトルストアのDBパス"),
        indexName: z.string().describe("ベクトルストアのインデックス名"),
    }),
    outputSchema: z.object({
        success: z.boolean(),
        message: z.string(),
        filePath: z.string(),
        strategy: z.string(),
        chunkCount: z.number().optional(),
        fileType: z.string().optional(),
        metadata: z.any().optional(),
    }),
    execute: async ({ context }) => {
        const {
            filePath,
            strategy,
            chunkSize,
            overlap,
            separator,
            dbPath,
            indexName,
        } = context;

        try {
            // ファイルが存在するか確認
            try {
                await fs.access(filePath);
            } catch (error) {
                return {
                    success: false,
                    message: `ファイル ${filePath} が見つかりません。`,
                    filePath,
                    strategy,
                };
            }

            // ファイル内容を読み込む
            const fileContent = await fs.readFile(filePath, "utf-8");

            if (!fileContent) {
                return {
                    success: false,
                    message: `ファイル ${filePath} は空です。`,
                    filePath,
                    strategy,
                };
            }

            // ファイルタイプを判断
            const fileExt = path.extname(filePath).toLowerCase();
            let fileType = "text";
            let doc;

            // ファイル拡張子に基づいて適切なMDocumentインスタンスを作成
            switch (fileExt) {
                case ".md":
                case ".markdown":
                    fileType = "markdown";
                    doc = MDocument.fromMarkdown(fileContent);
                    break;
                case ".html":
                case ".htm":
                    fileType = "html";
                    doc = MDocument.fromHTML(fileContent);
                    break;
                case ".json":
                    fileType = "json";
                    doc = MDocument.fromJSON(fileContent);
                    break;
                default:
                    fileType = "text";
                    doc = MDocument.fromText(fileContent);
            }

            // チャンキング - extractMetadataのエラーを修正
            const chunks = await doc.chunk({
                strategy: strategy,
                size: chunkSize,
                overlap: overlap,
                separator: separator,
            });

            if (chunks.length === 0) {
                return {
                    success: false,
                    message: `ファイル ${filePath} からチャンクを抽出できませんでした。`,
                    filePath,
                    strategy,
                    fileType,
                };
            }

            // チャンク処理成功のログ（埋め込み処理なしの場合のフォールバック用）
            console.log(
                `ファイル ${filePath} から ${chunks.length} 個のチャンクを抽出しました`
            );

            try {
                // LibSQLVectorライブラリを使用してベクトルストアを初期化
                const vectorStore = new LibSQLVector({
                    connectionUrl: `file:${dbPath}`,
                });

                // 必要に応じてインデックスを作成
                await vectorStore.createIndex({
                    indexName: indexName,
                    dimension: 768, // Google text-embedding-004 の次元数
                });

                // 埋め込みベクトルを生成
                // Googleのモデルは一度に最大100個のリクエストしか処理できないため、バッチ処理を実装
                const batchSize = 100;
                let allEmbeddings = [];

                for (let i = 0; i < chunks.length; i += batchSize) {
                    const batchChunks = chunks.slice(i, i + batchSize);
                    console.log(
                        `バッチ処理: ${i + 1}〜${Math.min(i + batchSize, chunks.length)}/${chunks.length}チャンク`
                    );

                    const { embeddings } = await embedMany({
                        model: googleEmbeddingModel,
                        values: batchChunks.map((chunk) => chunk.text),
                    });

                    allEmbeddings.push(...embeddings);
                }

                // メタデータを準備
                const metadata = chunks.map((chunk) => ({
                    text: chunk.text,
                    filePath: filePath,
                    fileType: fileType,
                    ...(chunk.metadata || {}),
                }));

                // ベクトルストアに保存（こちらもバッチ処理）
                for (let i = 0; i < allEmbeddings.length; i += batchSize) {
                    const batchEmbeddings = allEmbeddings.slice(
                        i,
                        i + batchSize
                    );
                    const batchMetadata = metadata.slice(i, i + batchSize);

                    console.log(
                        `ベクトル保存: ${i + 1}〜${Math.min(i + batchSize, allEmbeddings.length)}/${allEmbeddings.length}ベクトル`
                    );

                    await vectorStore.upsert({
                        indexName: indexName,
                        vectors: batchEmbeddings,
                        metadata: batchMetadata,
                    });
                }

                return {
                    success: true,
                    message: `ファイル ${filePath} のチャンキングと埋め込みが完了しました。${chunks.length}個のチャンクを作成しました。`,
                    filePath,
                    strategy,
                    chunkCount: chunks.length,
                    fileType,
                    metadata: {
                        totalChunks: chunks.length,
                        averageChunkLength: Math.round(
                            chunks.reduce(
                                (sum, chunk) => sum + chunk.text.length,
                                0
                            ) / chunks.length
                        ),
                    },
                };
            } catch (embeddingError: any) {
                // 埋め込み処理が失敗した場合でもチャンキング結果を返す
                console.warn(
                    `埋め込み処理でエラーが発生しました: ${embeddingError.message}`
                );
                return {
                    success: true,
                    message: `ファイル ${filePath} のチャンキングは成功しましたが、埋め込み処理でエラーが発生しました: ${embeddingError.message}`,
                    filePath,
                    strategy,
                    chunkCount: chunks.length,
                    fileType,
                };
            }
        } catch (error: any) {
            return {
                success: false,
                message: `ファイル処理中にエラーが発生しました: ${error.message}`,
                filePath,
                strategy,
            };
        }
    },
});
```

このコードを見て「難しそう...」と思ったかもしれませんが、心配いりません！一緒に一歩ずつ理解していきましょう。このツールは実に魅力的な機能を提供しているんですよ！

## fileProcessorToolの仕組みを解説 🧩

### 1. ファイル形式の自動認識 🔍

このツールはファイルの拡張子を自動的に検出して、最適な処理方法を選択します：

```typescript
// ファイルタイプを判断
const fileExt = path.extname(filePath).toLowerCase();
let fileType = "text";
let doc;

// ファイル拡張子に基づいて適切なMDocumentインスタンスを作成
switch (fileExt) {
    case ".md":
    case ".markdown":
        fileType = "markdown";
        doc = MDocument.fromMarkdown(fileContent);
        break;
    case ".html":
    case ".htm":
        fileType = "html";
        doc = MDocument.fromHTML(fileContent);
        break;
    case ".json":
        fileType = "json";
        doc = MDocument.fromJSON(fileContent);
        break;
    default:
        fileType = "text";
        doc = MDocument.fromText(fileContent);
}
```

この部分のおかげで、マークダウン、HTML、JSON、通常のテキストなど、様々な形式のファイルをシームレスに処理できます。構造を理解しながら適切にチャンキングするから、単純なテキスト分割よりずっと賢く処理できるんです！

### 2. 柔軟なチャンキング戦略 ✂️

このツールの真髄は、テキストを賢くチャンキングする能力にあります：

```typescript
inputSchema: z.object({
    // ...
    strategy: z
        .enum([
            "recursive",
            "character",
            "token",
            "markdown",
            "html",
            "json",
            "latex",
        ])
        .default("recursive")
        .describe("チャンキング戦略"),
    chunkSize: z.number().default(512).describe("チャンクのサイズ"),
    overlap: z.number().default(50).describe("チャンク間の重複トークン数"),
    separator: z.string().default("\n").describe("区切り文字"),
    // ...
}),
```

使える戦略は多彩です：

- **recursive**: 再帰的にテキストを分割する汎用戦略
- **character**: 文字数で単純に分割
- **token**: トークン（単語のような単位）で分割
- **markdown/html/json/latex**: 各形式の構造を尊重して分割

チャンクサイズとオーバーラップも自由に設定できるので、どんなファイルにも最適な設定を選べます！

### 3. Google埋め込みモデルの活用 🔮

このツールは、Googleの埋め込みモデルを使ってテキストをベクトル化します：

```typescript
// 埋め込みベクトルを生成
// Googleのモデルは一度に最大100個のリクエストしか処理できないため、バッチ処理を実装
const batchSize = 100;
let allEmbeddings = [];

for (let i = 0; i < chunks.length; i += batchSize) {
    const batchChunks = chunks.slice(i, i + batchSize);
    console.log(
        `バッチ処理: ${i + 1}〜${Math.min(i + batchSize, chunks.length)}/${chunks.length}チャンク`
    );

    const { embeddings } = await embedMany({
        model: googleEmbeddingModel,
        values: batchChunks.map((chunk) => chunk.text),
    });

    allEmbeddings.push(...embeddings);
}
```

Googleの埋め込みモデルは高品質で、テキストの意味をしっかりとベクトル空間に変換してくれます。これにより、検索時の精度が大幅に向上します！

### 4. 効率的なバッチ処理 📦

大量のチャンクを処理する際に、APIの制限にぶつからないよう賢くバッチ処理を行っています：

```typescript
// 埋め込み生成のバッチ処理
for (let i = 0; i < chunks.length; i += batchSize) {
    // ...処理...
}

// ベクトルストアへの保存もバッチ処理
for (let i = 0; i < allEmbeddings.length; i += batchSize) {
    // ...処理...
}
```

これにより、数千、数万のチャンクでも安定して処理できます。システムに優しく、APIの利用制限も回避できるスマートな設計です！

### 5. LibSQLを使ったローカルベクトルストア 💽

このツールは軽量で高速なLibSQLをベクトルストアとして使用します：

```typescript
// LibSQLVectorライブラリを使用してベクトルストアを初期化
const vectorStore = new LibSQLVector({
    connectionUrl: `file:${dbPath}`,
});
```

SQLiteベースのLibSQLを使うことで、追加のデータベースサーバーを立てることなく、シンプルなファイルベースでベクトルデータを保存できます。これは開発環境やデプロイの手間を大幅に削減する素晴らしい選択です！

### 6. 豊富なメタデータの保存 📝

チャンクのテキストだけでなく、有用なメタデータも一緒に保存します：

```typescript
// メタデータを準備
const metadata = chunks.map((chunk) => ({
    text: chunk.text,
    filePath: filePath,
    fileType: fileType,
    ...(chunk.metadata || {}),
}));
```

これにより、検索時に「このチャンクはどのファイルから来たのか」「どのようなタイプのファイルなのか」といった情報をすぐに取得できます。

## fileProcessorToolの活用シーン 🌟

このツールは以下のような場面で特に威力を発揮します：

1. **大規模コードベースの理解**: 何万行ものコードを持つリポジトリでも、関連部分を即座に検索
2. **ドキュメント検索の強化**: READMEやドキュメントの内容を意味ベースで簡単に検索
3. **プロジェクト固有の知識獲得**: チームのナレッジベースをAIエージェントに取り込み、より具体的な回答を得る
4. **複数ファイルの横断検索**: ファイル間の関連性を理解して、より包括的な回答を生成
5. **コードの変更影響分析**: 変更がコードの他の部分にどのような影響を与えるかを素早く分析

このツールを使えば、AIエージェントのリポジトリ理解能力が格段に向上します。「このリポジトリでどのようにAPI認証を実装していますか？」のような質問に、具体的なコード例を引用して答えられるようになるんです！

## fileProcessorToolの実装ポイント 📌

このツールを実装する際のポイントをいくつか挙げておきます：

1. **エラーハンドリングの重視**: ファイルの存在確認、空ファイルチェック、埋め込み処理の例外処理など、堅牢性を確保
2. **バッチ処理の実装**: API制限に対応した効率的なバッチ処理で、大量データも安定処理
3. **ファイル形式の自動判別**: 拡張子ベースで適切な処理方法を選択し、ユーザーの手間を省く
4. **柔軟なチャンキングオプション**: 様々な戦略やパラメータを提供し、多様なユースケースに対応
5. **メタデータの充実**: 単なるテキストではなく、コンテキスト情報も豊富に保存

特に注目すべきは、チャンキングと埋め込み処理を分離しているため、埋め込み処理に失敗してもチャンキング結果は返せるという堅牢な設計です。これにより、部分的な成功も許容する柔軟なツールとなっています！

## ベクトル検索ツール (vectorQueryTool) 🔍✨

**ファイルパス**: `src/mastra/tools/rag/vectorQuery.ts`

fileProcessorToolでファイルをチャンキングしてベクトルデータベースに保存したら、次に必要なのは「検索」の機能です。vectorQueryToolは、ベクトルデータベースに対して意味ベースの検索を行い、質問に関連する最も適切なテキスト/コード片を取得します。これがRAG技術の「Retrieval（検索）」部分を担当するツールです！

```typescript
import { createTool } from "@mastra/core/tools";
import { z } from "zod";
import { LibSQLVector } from "@mastra/core/vector/libsql";
import { embed } from "ai";
import { googleEmbeddingModel } from "../../models";

/**
 * ベクトルデータベースからクエリを実行するツール
 */
export const vectorQueryTool = createTool({
    id: "vector-query",
    description:
        "ベクトルデータベースに対してクエリを実行し、関連するコード片を取得します",
    inputSchema: z.object({
        query: z.string().describe("検索クエリ"),
        dbPath: z
            .string()
            .default("vector_store.db")
            .describe("ベクトルストアのDBパス"),
        indexName: z.string().describe("ベクトルストアのインデックス名"),
    }),
    outputSchema: z.object({
        success: z.boolean(),
        message: z.string(),
        results: z
            .array(
                z.object({
                    text: z.string(),
                    filePath: z.string().optional(),
                    fileType: z.string().optional(),
                    similarity: z.number().optional(),
                    metadata: z.any().optional(),
                })
            )
            .optional(),
    }),
    execute: async ({ context }) => {
        const { query, dbPath, indexName } = context;

        try {
            // ベクトルストアを初期化
            const vectorStore = new LibSQLVector({
                connectionUrl: `file:${dbPath}`,
            });

            try {
                // クエリをベクトル化
                const { embedding } = await embed({
                    model: googleEmbeddingModel,
                    value: query,
                });

                // ベクトル検索を実行
                const searchResults = await vectorStore.query({
                    indexName: indexName,
                    queryVector: embedding,
                });

                if (!searchResults || searchResults.length === 0) {
                    return {
                        success: true,
                        message: "クエリに一致する結果が見つかりませんでした。",
                        results: [],
                    };
                }

                // 結果を整形
                const formattedResults = searchResults.map((result: any) => ({
                    text: result.metadata?.text || "",
                    filePath: result.metadata?.filePath || "",
                    fileType: result.metadata?.fileType || "",
                    similarity: result.similarity,
                    metadata: result.metadata || {},
                }));

                return {
                    success: true,
                    message: `${searchResults.length}件の結果が見つかりました。`,
                    results: formattedResults,
                };
            } catch (embeddingError: any) {
                // エンベディングエラーが発生した場合は、ダミーの結果を返す
                console.warn(
                    `エンベディング処理でエラーが発生しました: ${embeddingError.message}`
                );
                return {
                    success: false,
                    message: `エンベディング処理でエラーが発生しました: ${embeddingError.message}`,
                    results: [],
                };
            }
        } catch (error: any) {
            return {
                success: false,
                message: `検索中にエラーが発生しました: ${error.message}`,
                results: [],
            };
        }
    },
});
```

このツールは一見シンプルですが、その中に隠された魔法のような機能があります。単なるキーワード検索ではなく、質問の「意味」を理解して関連するコンテンツを見つけ出すのです！これがRAG技術の核心部分です。

## vectorQueryToolの仕組みを解説 🧠

### 1. 意味ベースの検索クエリ 🔮

このツールの魅力は、テキストベースのクエリを意味を保持したベクトルに変換する点にあります：

```typescript
// クエリをベクトル化
const { embedding } = await embed({
    model: googleEmbeddingModel,
    value: query,
});
```

これにより、「認証システムの実装方法は？」というクエリが、`auth`、`authenticate`、`login`などの様々な関連コード片とマッチするようになります。キーワードが完全に一致しなくても、意味的に関連するコンテンツを見つけられるんです！

### 2. 同じ埋め込みモデルの使用 🧩

fileProcessorToolと同じGoogleの埋め込みモデルを使うことで、一貫性のある検索結果が得られます：

```typescript
const { embedding } = await embed({
    model: googleEmbeddingModel,
    value: query,
});
```

これは非常に重要なポイントです。保存時と検索時で異なる埋め込みモデルを使うと、ベクトル空間の特性が変わってしまい、検索精度が大幅に低下する可能性があります。

### 3. シンプルなクエリインターフェース 📝

複雑な検索条件を指定せずとも、自然な質問文だけで検索できるシンプルなインターフェースです：

```typescript
inputSchema: z.object({
    query: z.string().describe("検索クエリ"),
    dbPath: z
        .string()
        .default("vector_store.db")
        .describe("ベクトルストアのDBパス"),
    indexName: z.string().describe("ベクトルストアのインデックス名"),
}),
```

「このプロジェクトの認証方法を教えて」「データベース接続はどうやって設定する？」など、自然な質問文をそのまま入力できます。

### 4. 情報豊富な検索結果 📊

単なるテキストだけでなく、ファイルパスや類似度など、充実したメタデータを含む検索結果を返します：

```typescript
// 結果を整形
const formattedResults = searchResults.map((result: any) => ({
    text: result.metadata?.text || "",
    filePath: result.metadata?.filePath || "",
    fileType: result.metadata?.fileType || "",
    similarity: result.similarity,
    metadata: result.metadata || {},
}));
```

これにより、AIモデルは「このコード片はどのファイルにあるのか」「このドキュメント部分はどれくらい質問に関連しているのか」を判断できます。

### 5. 堅牢なエラーハンドリング 🛡️

検索プロセスやベクトル化処理でエラーが発生した場合でも、グレースフルに対応します：

```typescript
try {
    // ベクトル検索の処理...
} catch (embeddingError: any) {
    // エンベディングエラーが発生した場合は、ダミーの結果を返す
    console.warn(
        `エンベディング処理でエラーが発生しました: ${embeddingError.message}`
    );
    return {
        success: false,
        message: `エンベディング処理でエラーが発生しました: ${embeddingError.message}`,
        results: [],
    };
}
```

これにより、一時的なAPIエラーなどが発生しても、システム全体がクラッシュすることなく、適切なエラーメッセージを返します。

## vectorQueryToolとfileProcessorToolの連携 🤝

ここで重要なのは、vectorQueryToolとfileProcessorToolが連携して動作する点です。処理の流れを図式化すると：

1. **fileProcessorTool**: ファイル→チャンク→ベクトル→データベース保存
2. **vectorQueryTool**: 質問→ベクトル→類似ベクトル検索→関連チャンク取得

この2つのツールが連携することで、初めてRAG（検索拡張生成）の基盤が完成します！AIエージェントはこれらのツールを使って：

1. まず大量のファイルを処理してベクトルデータベースを構築（事前準備）
2. ユーザーからの質問に対して関連コンテンツを検索（リアルタイム）
3. 検索結果を使って、知識に裏付けられた正確な回答を生成（回答生成）

という流れでユーザーの質問に答えられるようになります。

## vectorQueryToolの活用シーン 🌟

このツールは以下のような場面で活躍します：

1. **コードベースの探索**: 「このプロジェクトでどうやってユーザー認証を実装している？」といった質問への回答
2. **ドキュメントからの情報抽出**: 「このプロジェクトのデプロイ手順は？」などの質問への回答
3. **関連コードの発見**: 「このエラーに関連するコードはどこ？」といった具体的な問題解決
4. **知識の横断検索**: 異なるファイルやモジュールにまたがる関連情報の検索
5. **実装例の提示**: 「このプロジェクトでのAPI呼び出しの例を見せて」といった要求に対応

特に、大規模なコードベースやドキュメントの中から「意味的に関連する情報」を素早く見つけられる点が、このツールの最大の魅力です！

## vectorQueryToolの実装ポイント 📌

このツールを実装する際のポイントをいくつか挙げておきましょう：

1. **シンプルなインターフェース設計**: 複雑な検索オプションを隠蔽し、シンプルなクエリだけで利用可能に
2. **一貫した埋め込みモデルの使用**: 保存と検索で同じモデルを使用して、検索精度を最大化
3. **結果のメタデータの充実**: テキスト内容だけでなく、ファイルパスや類似度などの情報も提供
4. **エラーハンドリングの実装**: API障害などに対して堅牢に動作するよう設計
5. **単一責任の原則**: 検索機能のみに集中し、他の処理（チャンキングなど）は別ツールに委譲

このツールの実装を通じて、AIエージェントに「記憶」と「検索」の能力を与えることができます。これにより、プロジェクト固有の知識に基づいた回答が可能になり、汎用AIを超えた専門的なアシスタンスが実現します！

## RAGツールの活用方法 🚀

ここまでで、RAG（検索拡張生成）の基盤となる2つの重要なツールを実装しました：

1. **fileProcessorTool**: ファイルをチャンキングしてベクトル化、保存する「知識獲得」ツール
2. **vectorQueryTool**: 自然言語の質問から関連情報を検索する「知識検索」ツール

これらのツールをMastraエージェントに組み込むことで、プロジェクト固有の知識を持ったAIアシスタントが実現します。例えば：

```typescript
// GitHub解析エージェントの例
export const githubAnalysisAgent = new Agent({
    name: "GitHub Analysis Agent",
    instructions: `あなたはリポジトリを深く理解するGitHub分析エキスパートです。
クローンされたリポジトリの構造を分析し、コードの理解を深めるために以下のツールを使用します：
1. ファイル処理ツール: コードファイルをチャンキングしてベクトルデータベースに保存
2. ベクトル検索ツール: ユーザーの質問に関連するコード片を検索
具体的なコード例を挙げながら、わかりやすく説明してください。`,
    model: google("gemini-2.0-flash-001"),
    tools: {
        fileProcessorTool,
        vectorQueryTool,
    },
});
```

このようなエージェントを作成することで、ユーザーはリポジトリについて自然な言葉で質問でき、AIは関連するコード例を引用しながら具体的な回答を提供できるようになります。

RAG技術は確かに複雑ですが、Mastraフレームワークを使えば、このような高度な機能も簡単に実装できるのです！ここで学んだ知識を活かして、あなた自身のスマートなAIエージェントを作ってみてください。きっと素晴らしい体験が待っていますよ！ 