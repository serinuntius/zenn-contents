---
title: "第8章：チートシート生成ツール（tools/cheatsheet）"
---

# チートシート生成と保存ツール 📝✨

プログラミングやAI開発の世界では、知識の整理と素早い参照がとても重要です。そこで登場するのがチートシートツール！複雑な情報を整理して、いつでも参照できるようにするための強力な味方です。このチャプターでは、AIエージェントが生成したチートシートを簡単に保存できるツールの実装方法を学びましょう！

## チートシート保存ツールの概要 🗂️

**ファイルパス**: `src/mastra/tools/cheatsheet/saveCheatsheet.ts`

チートシートは開発者の強い味方です。コマンド一覧、APIリファレンス、設定オプション、ベストプラクティス...覚えきれない情報を整理して、必要なときにパッと見られるようにするものです。そんなチートシートをAIに生成してもらい、ファイルとして保存できれば便利ですよね？

今回紹介する`saveCheatsheetTool`は、AIが生成したチートシートを簡単にファイルに保存するためのツールです。単に保存するだけでなく、追記モードやセクション分割にも対応しているんですよ！

## ツールの実装 🛠️

早速コードを見てみましょう：

```typescript
import { createTool } from "@mastra/core/tools";
import { z } from "zod";
import fs from "fs/promises";
import path from "path";

/**
 * チートシート保存操作の結果を表すスキーマ
 */
export const saveCheatsheetOutputSchema = z
    .object({
        success: z.boolean().describe("保存操作が成功したかどうか"),
        message: z.string().describe("操作結果の詳細メッセージ"),
        filePath: z.string().describe("保存されたファイルのパス"),
        filename: z.string().describe("ファイル名"),
        bytesWritten: z.number().optional().describe("書き込まれたバイト数"),
        isContinuation: z.boolean().optional().describe("継続書き込みかどうか"),
        contentLength: z.number().optional().describe("コンテンツの長さ"),
    })
    .describe("チートシート保存操作の結果");

/**
 * チートシートを保存するツール
 */
export const saveCheatsheetTool = createTool({
    id: "save-cheatsheet",
    description: "生成されたチートシートをファイルに保存します",
    inputSchema: z.object({
        content: z.string().describe("保存するチートシートの内容"),
        outputPath: z.string().describe("保存先のファイルパス"),
        append: z
            .boolean()
            .default(false)
            .describe("既存ファイルに追記する場合はtrue"),
        section: z
            .string()
            .optional()
            .describe("セクション名（複数セクションに分割する場合）"),
        sectionIndex: z
            .number()
            .optional()
            .describe("セクションのインデックス"),
        totalSections: z.number().optional().describe("総セクション数"),
    }),
    outputSchema: saveCheatsheetOutputSchema,
    execute: async ({ context }) => {
        const { content, outputPath, append, section } = context;

        try {
            // 出力ディレクトリが存在することを確認
            const dirPath = path.dirname(outputPath);
            try {
                await fs.mkdir(dirPath, { recursive: true });
            } catch (error) {
                // ディレクトリ作成エラーを無視（既に存在する場合など）
            }

            // ファイルへの書き込みフラグ
            const flag = append ? "a" : "w";

            // セクションヘッダーを追加（指定されている場合）
            let contentToWrite = content;
            if (section && !append) {
                contentToWrite = `# ${section}\n\n${content}`;
            } else if (section && append) {
                contentToWrite = `\n\n# ${section}\n\n${content}`;
            }

            // ファイルに書き込み
            await fs.writeFile(outputPath, contentToWrite, { flag });

            // ファイル情報の取得
            const stats = await fs.stat(outputPath);

            return {
                success: true,
                message: append
                    ? `チートシートを ${outputPath} に追記しました${section ? ` (セクション: ${section})` : ""}`
                    : `チートシートを ${outputPath} に保存しました${section ? ` (セクション: ${section})` : ""}`,
                filePath: outputPath,
                filename: path.basename(outputPath),
                bytesWritten: Buffer.byteLength(contentToWrite, "utf8"),
                isContinuation: append,
                contentLength: stats.size,
            };
        } catch (error: any) {
            return {
                success: false,
                message: `チートシートの保存中にエラーが発生しました: ${error.message}`,
                filePath: outputPath,
                filename: path.basename(outputPath),
            };
        }
    },
});
```

このコードはサクッと実装できていますが、実際にはかなり便利な機能が詰まっています！順番に解説していきましょう。

## 入力と出力のスキーマ定義 📋

Zodを使った明確なスキーマ定義が、このツールの使いやすさを支えています：

### 入力スキーマ

```typescript
inputSchema: z.object({
    content: z.string().describe("保存するチートシートの内容"),
    outputPath: z.string().describe("保存先のファイルパス"),
    append: z
        .boolean()
        .default(false)
        .describe("既存ファイルに追記する場合はtrue"),
    section: z
        .string()
        .optional()
        .describe("セクション名（複数セクションに分割する場合）"),
    sectionIndex: z
        .number()
        .optional()
        .describe("セクションのインデックス"),
    totalSections: z.number().optional().describe("総セクション数"),
}),
```

この入力スキーマがとても柔軟で、様々なユースケースに対応できるんです！

- **content**: 保存したいチートシートの内容そのものです。Markdownフォーマットが一般的ですが、任意のテキスト形式で保存できます。
- **outputPath**: 出力先のファイルパスです。相対パスでも絶対パスでもOK！
- **append**: 既存のファイルに追記するかどうかのフラグです。これがあることで、チートシートを少しずつ生成して追加していけます。
- **section**: オプショナルなセクション名です。チートシートを複数のセクションに分けたい場合に使います。
- **sectionIndex** と **totalSections**: 大きなチートシートを分割生成する場合の位置情報です。

### 出力スキーマ

```typescript
outputSchema: z.object({
    success: z.boolean().describe("保存操作が成功したかどうか"),
    message: z.string().describe("操作結果の詳細メッセージ"),
    filePath: z.string().describe("保存されたファイルのパス"),
    filename: z.string().describe("ファイル名"),
    bytesWritten: z.number().optional().describe("書き込まれたバイト数"),
    isContinuation: z.boolean().optional().describe("継続書き込みかどうか"),
    contentLength: z.number().optional().describe("コンテンツの長さ"),
}),
```

出力スキーマも情報が豊富で、保存操作の結果を詳細に把握できます：

- **success**: 操作が成功したかどうかの基本フラグ
- **message**: 人間が読みやすい結果メッセージ
- **filePath** と **filename**: 保存先の情報
- **bytesWritten**: 実際に書き込まれたバイト数（デバッグや進捗管理に便利）
- **contentLength**: 最終的なファイルサイズ

## 実行ロジック 🧩

実行部分のコードも見ていきましょう：

```typescript
execute: async ({ context }) => {
    const { content, outputPath, append, section } = context;

    try {
        // 出力ディレクトリが存在することを確認
        const dirPath = path.dirname(outputPath);
        try {
            await fs.mkdir(dirPath, { recursive: true });
        } catch (error) {
            // ディレクトリ作成エラーを無視（既に存在する場合など）
        }

        // ファイルへの書き込みフラグ
        const flag = append ? "a" : "w";

        // セクションヘッダーを追加（指定されている場合）
        let contentToWrite = content;
        if (section && !append) {
            contentToWrite = `# ${section}\n\n${content}`;
        } else if (section && append) {
            contentToWrite = `\n\n# ${section}\n\n${content}`;
        }

        // ファイルに書き込み
        await fs.writeFile(outputPath, contentToWrite, { flag });

        // ファイル情報の取得
        const stats = await fs.stat(outputPath);

        return {
            success: true,
            message: append
                ? `チートシートを ${outputPath} に追記しました${section ? ` (セクション: ${section})` : ""}`
                : `チートシートを ${outputPath} に保存しました${section ? ` (セクション: ${section})` : ""}`,
            filePath: outputPath,
            filename: path.basename(outputPath),
            bytesWritten: Buffer.byteLength(contentToWrite, "utf8"),
            isContinuation: append,
            contentLength: stats.size,
        };
    } catch (error: any) {
        return {
            success: false,
            message: `チートシートの保存中にエラーが発生しました: ${error.message}`,
            filePath: outputPath,
            filename: path.basename(outputPath),
        };
    }
}
```

このロジックには便利な機能がいくつも含まれています：

### 1. ディレクトリの自動作成 📁

```typescript
// 出力ディレクトリが存在することを確認
const dirPath = path.dirname(outputPath);
try {
    await fs.mkdir(dirPath, { recursive: true });
} catch (error) {
    // ディレクトリ作成エラーを無視（既に存在する場合など）
}
```

指定したパスのディレクトリが存在しなくても、自動的に作成してくれます！`recursive: true`オプションのおかげで、複数階層のディレクトリも一気に作れるんですよ。

### 2. 追記モードのサポート 📝

```typescript
// ファイルへの書き込みフラグ
const flag = append ? "a" : "w";
```

`append`フラグが`true`の場合は既存ファイルに追記し、`false`の場合は上書きします。この機能があれば、チートシートを少しずつ増やしていけますね！

### 3. セクション自動フォーマット 📑

```typescript
// セクションヘッダーを追加（指定されている場合）
let contentToWrite = content;
if (section && !append) {
    contentToWrite = `# ${section}\n\n${content}`;
} else if (section && append) {
    contentToWrite = `\n\n# ${section}\n\n${content}`;
}
```

`section`が指定されていると、自動的にMarkdownの見出しとして追加されます。追記モードの場合は適切な改行も入れてくれるので、綺麗な形式を保てます。

### 4. 詳細な結果情報 📊

```typescript
return {
    success: true,
    message: append
        ? `チートシートを ${outputPath} に追記しました${section ? ` (セクション: ${section})` : ""}`
        : `チートシートを ${outputPath} に保存しました${section ? ` (セクション: ${section})` : ""}`,
    filePath: outputPath,
    filename: path.basename(outputPath),
    bytesWritten: Buffer.byteLength(contentToWrite, "utf8"),
    isContinuation: append,
    contentLength: stats.size,
};
```

単に成功/失敗だけでなく、書き込まれたバイト数やファイルサイズなど、詳細な情報を返してくれます。これはAIエージェントが処理を正確に追跡するのに役立ちます。

## チートシート保存ツールの活用シーン 💡

このツールは様々なシーンで活躍します：

### 1. 技術リファレンスの作成

AIに特定の技術や言語のチートシートを生成させ、いつでも参照できるように保存：

```typescript
// Pythonの基本文法についてのチートシートを生成し保存
const pythonBasicResponse = await agent.generate(
    "Python基本文法についてのチートシートを作成し、./docs/cheatsheets/python.md に保存してください。"
);

// 後で別のセクションを追加
const pythonAdvancedResponse = await agent.generate(
    "Python高度な機能についてのチートシートを作成し、既存の./docs/cheatsheets/python.mdファイルに追加してください。"
);
```

### 2. プロジェクト固有の情報整理

特定のプロジェクトやAPIの使い方をまとめて、チーム全体で共有：

```typescript
const apiCheatsheetResponse = await agent.generate(
    "私たちのプロジェクトのAPI仕様をチートシート形式でまとめて、./docs/project-api-cheatsheet.mdに保存してください。"
);
```

### 3. 大規模なチートシートの分割生成

トークン制限のあるAIモデルでも、大きなチートシートを分割して生成：

```typescript
// 3つのセクションに分けて生成
const sections = ["基本文法", "中級テクニック", "高度な型システム"];

for (let i = 0; i < sections.length; i++) {
    const isFirstSection = i === 0;
    
    const prompt = `TypeScriptの${sections[i]}に関するチートシートを作成し、` +
        `./docs/typescript-cheatsheet.mdに${isFirstSection ? '保存' : '追加'}してください。` +
        `これは${i+1}/${sections.length}番目のセクションです。`;
    
    const response = await agent.generate(prompt);
    
    console.log(`${sections[i]}セクションを${isFirstSection ? '作成' : '追加'}しました。`);
}
```

## エージェント連携の例 🤖

チートシート保存ツールを組み込んだMastraエージェントの例を見てみましょう：

```typescript
import { Agent } from "@mastra/core";
import { saveCheatsheetTool } from "./tools/cheatsheet/saveCheatsheet";
import { google } from "@mastra/providers";

export const cheatsheetAgent = new Agent({
    name: "チートシート生成エージェント",
    instructions: `あなたは開発者向けのチートシート生成スペシャリストです。
    ユーザーの要求に応じて、簡潔で実用的なチートシートを生成し、ファイルに保存します。
    チートシートは常にマークダウン形式で生成し、以下の要素を含めるよう心がけてください：
    
    1. 簡潔なタイトルと説明
    2. カテゴリー別の整理
    3. コード例（適切な場合）
    4. 実用的なヒントやベストプラクティス
    
    大きなトピックの場合は、複数のセクションに分けて生成することも検討してください。`,
    model: google("gemini-2.0-flash-001"),
    tools: {
        saveCheatsheetTool,
    },
});
```

このエージェントは、ユーザーの要求に応じてチートシートを生成し、適切なファイルに保存する役割を担います。例えば：

```typescript
const response = await cheatsheetAgent.generate(
    "Node.jsのファイル操作について、初心者向けのチートシートを作成して保存してください。"
);
```

このプロンプトに対して、エージェントは：

1. Node.jsのファイル操作に関するチートシートを生成
2. 適切なファイル名とパスを決定
3. `saveCheatsheetTool`を使って保存
4. 保存結果をユーザーに報告

という一連の流れを自動的に実行します。

## 実装のポイント 📌

このツールを実装する際の重要なポイントをまとめておきましょう：

1. **フォルダ自動作成**: ユーザーが指定したパスのディレクトリが存在しない場合でも、自動的に作成する機能で使いやすさが向上
2. **追記モード**: 一度に生成しきれない大きなチートシートも、複数回に分けて追記できる柔軟性
3. **セクション管理**: チートシートを論理的なセクションに分割して整理できる機能
4. **エラーハンドリング**: ファイルシステム操作に関する様々なエラーに対応する堅牢性
5. **詳細な結果情報**: 単なる成功/失敗だけでなく、具体的な情報を返すことで後続処理に役立つ

## Mastra RAGツールとの連携 🔄

前回のチャプターで学んだRAGツールと組み合わせると、さらに強力なチートシート生成が可能になります：

```typescript
import { Agent } from "@mastra/core";
import { fileProcessorTool, vectorQueryTool } from "./tools/rag";
import { saveCheatsheetTool } from "./tools/cheatsheet/saveCheatsheet";
import { google } from "@mastra/providers";

export const smartCheatsheetAgent = new Agent({
    name: "スマートチートシート生成エージェント",
    instructions: `あなたはプロジェクト固有のチートシート生成スペシャリストです。
    1. まずプロジェクトのコードやドキュメントを処理してベクトルDBに保存
    2. 関連情報をベクトル検索で取得
    3. 取得した情報を元に、実用的なチートシートを生成
    4. 生成したチートシートをファイルに保存
    という流れで、プロジェクト固有の知識に基づくチートシートを作成します。`,
    model: google("gemini-2.0-flash-001"),
    tools: {
        fileProcessorTool,
        vectorQueryTool,
        saveCheatsheetTool,
    },
});
```

このエージェントを使えば、例えばプロジェクトのソースコードやドキュメントを分析して、そのプロジェクト特有のAPIやコマンドのチートシートを自動生成できます！

## チートシート生成でよくある問題と解決法 🔍

### 1. コンテンツが長すぎる場合

AIモデルの出力制限を超えるような長いチートシートを生成したい場合：

```typescript
// トピックを複数の小さなチャンクに分割
const topics = ["基本構文", "データ型", "関数", "クラス", "例外処理", "モジュール"];

for (let i = 0; i < topics.length; i++) {
    const prompt = `Pythonの${topics[i]}についてのチートシートを生成してください。`;
    
    const response = await cheatsheetAgent.generate(prompt);
    
    await cheatsheetAgent.runTool("saveCheatsheetTool", {
        content: response.text,
        outputPath: "./cheatsheets/python_complete.md",
        section: topics[i],
        append: i > 0,
    });
}
```

### 2. 既存チートシートの更新

プロジェクトの更新に合わせてチートシートも更新したい場合：

```typescript
// 既存のチートシートを読み込む
const existingContent = await fs.readFile("./cheatsheets/api_reference.md", "utf-8");

// 更新用のプロンプト
const prompt = `以下は既存のAPIリファレンスチートシートです：

${existingContent}

プロジェクトに新しく追加された以下のAPIエンドポイントも含めて、チートシートを更新してください：
- GET /api/v2/users/recommendations
- POST /api/v2/content/share
`;

const response = await cheatsheetAgent.generate(prompt);

// 更新されたチートシートを保存
await cheatsheetAgent.runTool("saveCheatsheetTool", {
    content: response.text,
    outputPath: "./cheatsheets/api_reference_updated.md",
});
```

## まとめ 🎯

Mastraフレームワークの`saveCheatsheetTool`は、AIが生成した知識を整理して保存するための強力なツールです。シンプルなAPIながら、追記モードやセクション管理、フォルダ自動作成など、実用的な機能が詰まっています。

RAGツールと組み合わせれば、プロジェクト固有の知識に基づいたカスタムチートシートの自動生成も可能になります。これにより、開発者は複雑な情報を整理し、いつでも参照できる形で保存できるのです。

チートシートは学習の強力な味方です。複雑な概念や構文を簡潔にまとめることで、開発の効率化と知識の定着に大いに役立ちます。このツールを活用して、あなたの開発ライフをもっと便利にしてみませんか？

次のチャプターでは、これまで学んだツールやエージェントを組み合わせて、より複雑な実用的なシステムを構築する方法を探っていきましょう。Mastraフレームワークの可能性は無限大です！ 