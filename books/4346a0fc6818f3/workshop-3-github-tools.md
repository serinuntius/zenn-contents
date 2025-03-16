---
title: "第5章：GitHubリポジトリの操作（tools/github）"
---

# GitHubリポジトリの操作方法を学ぼう！🚀

前章ではAIモデルの設定と選択について学びました。エージェントの「頭脳」が準備できたところで、次は「手足」となる部分—つまりツール（Tools）の作成に進みましょう！

特に今回は、GitHubリポジトリを操作するツールを作成します。これは私たちのエージェントが外部のコードリポジトリにアクセスして分析できるようにするための第一歩です。さあ、早速実装していきましょう！✨

## ツールとは？エージェントの能力を拡張する魔法の道具 🧰

Mastraフレームワークにおける「ツール」とは、AIエージェントが外部のサービスやシステムと対話するための機能のことです。ツールがあることで、エージェントは単なる会話だけでなく、実際のアクションを実行できるようになります。

GitHubリポジトリをクローンするツールを作ることで、エージェントは実際のコードベースにアクセスし、分析することが可能になります。これはとても強力な機能ですよね！

## GitHubリポジトリクローンツールの実装 👨‍💻

まずは、GitHubリポジトリをクローンするツールのコードを見てみましょう。このコードは`src/mastra/tools/github-clone-tool.ts`として保存します：

```typescript
import { createTool } from "@mastra/core/tools";
import { z } from "zod";
import { exec } from "child_process";
import { promisify } from "util";
import path from "path";
import fs from "fs";

const execAsync = promisify(exec);

/**
 * クローン操作の結果を表すスキーマ
 */
export const cloneOutputSchema = z
    .object({
        success: z.boolean().describe("クローン操作が成功したかどうか"),
        message: z.string().describe("操作結果の詳細メッセージ"),
        repositoryFullPath: z
            .string()
            .optional()
            .describe("クローンされたリポジトリの絶対パス（フルパス）"),
        cloneDirectoryName: z
            .string()
            .optional()
            .describe("クローン先のディレクトリ名（相対パス）"),
    })
    .describe("リポジトリクローン操作の結果");

/**
 * GitHub リポジトリをクローンするツール
 * LFS対応とサブモジュール処理も可能
 */
export const cloneRepositoryTool = createTool({
    id: "clone-repository",
    description: "GitHub リポジトリをクローンします",
    inputSchema: z.object({
        repositoryUrl: z
            .string()
            .describe("リポジトリのURL（https://github.com/user/repo 形式）"),
        branch: z
            .string()
            .optional()
            .describe(
                "クローンするブランチ（指定しない場合はデフォルトブランチ）"
            ),
        includeLfs: z
            .boolean()
            .optional()
            .default(false)
            .describe("Git LFSファイルも取得するか"),
        includeSubmodules: z
            .boolean()
            .optional()
            .default(false)
            .describe("サブモジュールも取得するか"),
    }),
    outputSchema: cloneOutputSchema,
    execute: async ({ context }) => {
        const { repositoryUrl, branch, includeLfs, includeSubmodules } =
            context;

        try {
            // リポジトリ名を取得
            const repoName =
                repositoryUrl.split("/").pop()?.replace(".git", "") || "repo";
            const cloneDir = repoName;
            const fullPath = path.resolve(process.cwd(), cloneDir);

            // ディレクトリが既に存在するか確認
            if (fs.existsSync(fullPath)) {
                return {
                    success: true,
                    message: `ディレクトリ ${cloneDir} は既に存在するため、クローンをスキップしました。`,
                    repositoryFullPath: fullPath,
                    cloneDirectoryName: cloneDir,
                };
            }

            // クローンコマンドを構築
            let command = `git clone ${repositoryUrl}`;

            // ブランチが指定されている場合
            if (branch) {
                command += ` -b ${branch}`;
            }

            // サブモジュールが必要な場合
            if (includeSubmodules) {
                command += ` --recurse-submodules`;
            }

            // ターゲットディレクトリを指定
            command += ` ${cloneDir}`;

            // コマンド実行
            const { stdout, stderr } = await execAsync(command);

            // LFSファイルが必要な場合
            if (includeLfs) {
                try {
                    // ディレクトリに移動してLFSファイルを取得
                    await execAsync(`cd ${cloneDir} && git lfs pull`);
                } catch (error: any) {
                    return {
                        success: true,
                        message: `リポジトリのクローンは成功しましたが、LFSファイルの取得に失敗しました: ${error.message}`,
                        repositoryFullPath: fullPath,
                        cloneDirectoryName: cloneDir,
                    };
                }
            }

            return {
                success: true,
                message: `リポジトリを ${fullPath} にクローンしました。`,
                repositoryFullPath: fullPath,
                cloneDirectoryName: cloneDir,
            };
        } catch (error: any) {
            // ダミーの値として、プロセスの作業ディレクトリと"repo"を返す
            const dummyCloneDir = "repo";
            const dummyFullPath = path.resolve(process.cwd(), dummyCloneDir);

            console.error(`クローンエラー: ${error.message}`);
            console.error(
                `デバッグ情報: リポジトリURL=${repositoryUrl}, ブランチ=${branch || "default"}`
            );

            return {
                success: false,
                message: `クローンに失敗しました: ${error.message}`,
                repositoryFullPath: dummyFullPath,
                cloneDirectoryName: dummyCloneDir,
            };
        }
    },
});
```

一見するとコードが長く感じるかもしれませんが、一つずつ解説していくと意外とシンプルです。それでは、このコードを詳しく見ていきましょう！

## ツール作成の基本要素を理解しよう 🧩

### 1. 必要なパッケージのインポート 📦

```typescript
import { createTool } from "@mastra/core/tools";
import { z } from "zod";
import { exec } from "child_process";
import { promisify } from "util";
import path from "path";
import fs from "fs";
```

まず必要なパッケージをインポートしています：
- `createTool`: Mastraフレームワークでツールを作成するための関数
- `z`（Zod）: 入力と出力のスキーマを定義するためのライブラリ
- `exec`, `promisify`: Gitコマンドを実行するための関数
- `path`, `fs`: ファイルシステム操作のためのNode.jsモジュール

### 2. 出力スキーマの定義 📝

```typescript
export const cloneOutputSchema = z
    .object({
        success: z.boolean().describe("クローン操作が成功したかどうか"),
        message: z.string().describe("操作結果の詳細メッセージ"),
        repositoryFullPath: z
            .string()
            .optional()
            .describe("クローンされたリポジトリの絶対パス（フルパス）"),
        cloneDirectoryName: z
            .string()
            .optional()
            .describe("クローン先のディレクトリ名（相対パス）"),
    })
    .describe("リポジトリクローン操作の結果");
```

Zodを使用して、ツールの出力データの形式を定義しています。これにより：
1. AIモデルが出力の構造を理解できる
2. 型安全性が保証される
3. 自動ドキュメント生成が可能になる

### 3. ツールの作成と入力スキーマの定義 🛠️

```typescript
export const cloneRepositoryTool = createTool({
    id: "clone-repository",
    description: "GitHub リポジトリをクローンします",
    inputSchema: z.object({
        repositoryUrl: z
            .string()
            .describe("リポジトリのURL（https://github.com/user/repo 形式）"),
        branch: z
            .string()
            .optional()
            .describe(
                "クローンするブランチ（指定しない場合はデフォルトブランチ）"
            ),
        includeLfs: z
            .boolean()
            .optional()
            .default(false)
            .describe("Git LFSファイルも取得するか"),
        includeSubmodules: z
            .boolean()
            .optional()
            .default(false)
            .describe("サブモジュールも取得するか"),
    }),
    outputSchema: cloneOutputSchema,
    // executeは後で説明
```

ここでは、`createTool`関数を使用してツールを定義しています。主なプロパティは：

- `id`: ツールの一意の識別子
- `description`: ツールの簡単な説明（AIモデルがこれを読んで使い方を理解）
- `inputSchema`: 入力パラメータの形式を定義したZodスキーマ
- `outputSchema`: 先ほど定義した出力形式

入力スキーマでは、GitHubリポジトリのURLを必須とし、ブランチ、LFSファイル取得、サブモジュール取得をオプションとしています。これはツールに柔軟性を持たせるための設計です。

### 4. ツールのロジック実装 💻

```typescript
execute: async ({ context }) => {
    const { repositoryUrl, branch, includeLfs, includeSubmodules } =
        context;

    try {
        // リポジトリ名を取得
        const repoName =
            repositoryUrl.split("/").pop()?.replace(".git", "") || "repo";
        const cloneDir = repoName;
        const fullPath = path.resolve(process.cwd(), cloneDir);

        // ディレクトリが既に存在するか確認
        if (fs.existsSync(fullPath)) {
            return {
                success: true,
                message: `ディレクトリ ${cloneDir} は既に存在するため、クローンをスキップしました。`,
                repositoryFullPath: fullPath,
                cloneDirectoryName: cloneDir,
            };
        }

        // クローンコマンドを構築
        let command = `git clone ${repositoryUrl}`;

        // ブランチが指定されている場合
        if (branch) {
            command += ` -b ${branch}`;
        }

        // サブモジュールが必要な場合
        if (includeSubmodules) {
            command += ` --recurse-submodules`;
        }

        // ターゲットディレクトリを指定
        command += ` ${cloneDir}`;

        // コマンド実行
        const { stdout, stderr } = await execAsync(command);

        // LFSファイルが必要な場合
        if (includeLfs) {
            try {
                // ディレクトリに移動してLFSファイルを取得
                await execAsync(`cd ${cloneDir} && git lfs pull`);
            } catch (error: any) {
                return {
                    success: true,
                    message: `リポジトリのクローンは成功しましたが、LFSファイルの取得に失敗しました: ${error.message}`,
                    repositoryFullPath: fullPath,
                    cloneDirectoryName: cloneDir,
                };
            }
        }

        return {
            success: true,
            message: `リポジトリを ${fullPath} にクローンしました。`,
            repositoryFullPath: fullPath,
            cloneDirectoryName: cloneDir,
        };
    } catch (error: any) {
        // エラーハンドリング部分は省略
    }
}
```

`execute`関数は、入力パラメータに基づいてリポジトリをクローンするメインのロジックです。以下のステップで処理を行います：

1. URLからリポジトリ名を抽出
2. 既にディレクトリが存在する場合はスキップ
3. 適切なGitコマンドを構築（ブランチやサブモジュールのオプションを含む）
4. コマンドを実行してリポジトリをクローン
5. 必要に応じてGit LFSファイルを取得
6. 結果を返す（成功または失敗）

### 5. エラーハンドリングのベストプラクティス 🛡️

```typescript
catch (error: any) {
    // ダミーの値として、プロセスの作業ディレクトリと"repo"を返す
    const dummyCloneDir = "repo";
    const dummyFullPath = path.resolve(process.cwd(), dummyCloneDir);

    console.error(`クローンエラー: ${error.message}`);
    console.error(
        `デバッグ情報: リポジトリURL=${repositoryUrl}, ブランチ=${branch || "default"}`
    );

    return {
        success: false,
        message: `クローンに失敗しました: ${error.message}`,
        repositoryFullPath: dummyFullPath,
        cloneDirectoryName: dummyCloneDir,
    };
}
```

エラーハンドリングは、堅牢なツールを作成する上で非常に重要です。このコードでは：

1. エラーが発生した場合でも、定義したスキーマに合った値を返す
2. デバッグ情報をログに出力してトラブルシューティングを容易にする
3. エラーメッセージを含んだ結果を返してユーザーに何が起きたかを伝える

## 実践：ツールの使い方 🚀

さて、このツールをどのように使用するのでしょうか？Mastraエージェントでは以下のように組み込むことができます：

```typescript
import { Agent } from "@mastra/core/agent";
import { createGoogleGenerativeAI } from "@ai-sdk/google";
import { cloneRepositoryTool } from "../tools/github-clone-tool";

// Google Gemini AIプロバイダーの作成
const google = createGoogleGenerativeAI({
    apiKey: process.env.GOOGLE_API_KEY || "",
});

// エージェント定義
const githubAnalysisAgent = new Agent({
  name: "GitHub Analysis Agent",
  instructions: "GitHubリポジトリを解析するエージェントです。リポジトリのURLを指定すると、それをクローンして解析できます。",
  model: google("gemini-2.0-flash-001"),
  tools: {
    cloneRepositoryTool
  }
});
```

このように、作成したツールをエージェントの`tools`プロパティに登録するだけで、AIモデルはそのツールを使用できるようになります。AIモデルは与えられた指示とツールの説明を基に、適切なタイミングでツールを呼び出すことを学習します。

## Zodスキーマの力 - わかりやすいAPI設計の秘訣 💪

ここで、Zodスキーマについてもう少し詳しく説明しましょう。Zodは型安全なスキーマ検証ライブラリで、以下の利点があります：

1. **AIモデルの理解を助ける**: 各フィールドに`.describe()`で説明を追加することで、AIモデルがツールの使い方を理解しやすくなります
2. **バリデーション**: 入力値が期待通りの形式かを自動的に検証
3. **型安全性**: TypeScriptと統合されるため、開発時のエラーを減少
4. **ドキュメント生成**: スキーマからAPIドキュメントを自動生成可能

例えば、リポジトリURLのスキーマ定義：

```typescript
repositoryUrl: z
    .string()
    .describe("リポジトリのURL（https://github.com/user/repo 形式）")
```

こうした説明文はAIモデルに「このフィールドには何を入れるべきか」を教える重要な情報となります。

## ツール設計のベストプラクティス 🏆

GitHubリポジトリクローンツールから学べる設計のベストプラクティスをまとめてみましょう：

1. **明確な入出力定義**: Zodを使って入力と出力を明確に定義
2. **適切な粒度**: 一つの機能に特化したツールを作る（単一責任の原則）
3. **堅牢なエラーハンドリング**: 失敗しても適切な情報を返す
4. **オプションのサポート**: 必要に応じて機能を拡張できるオプションパラメータ
5. **詳細なログ記録**: デバッグに役立つ情報をログに残す

これらの原則に従うことで、再利用性が高く、メンテナンスしやすいツールを作成できます。

## ここまでのまとめ 📝

この章では、GitHubリポジトリをクローンするツールを例に、Mastraフレームワークでのツール作成方法を学びました：

1. ツールの基本構造とその要素
2. Zodを使った入出力スキーマの定義
3. 実際のロジック実装と外部コマンド（Git）の実行
4. エラーハンドリングのベストプラクティス
5. エージェントへのツール組み込み方法

これで、エージェントはGitHubリポジトリをクローンできるようになりました。次の章では、クローンしたリポジトリを解析するためのツールを作成していきます。

エージェントの「手足」が一つ増えましたね！次は、クローンしたコードベースを読み解くための「目」となるコード解析ツールを実装していきましょう。お楽しみに！🚀 