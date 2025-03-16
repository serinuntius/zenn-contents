---
title: "第6章：コード解析ツールの実装（tools/analysis）"
---

# コード解析ツールの実装方法を学ぼう！🔍

前章ではGitHubリポジトリをクローンする方法を学びました。素晴らしい！これでエージェントはコードリポジトリにアクセスできるようになりました。でも、リポジトリをクローンしただけでは、その中身を理解することはできません。次に必要なのは、クローンしたコードベースを解析する「目」の役割を果たすツールです。

この章では、3つの強力なコード解析ツールを順番に実装していきます。最初に紹介するのは「treeAnalyzerTool」です。これはリポジトリの構造を素早く把握するための強力な味方となります！💪

## リポジトリ構造解析ツール (treeAnalyzerTool) 🌳

**ファイルパス**: `src/mastra/tools/analysis/treeAnalyzer.ts`

コードベースを理解する第一歩は、その構造を把握することです。treeAnalyzerToolは、リポジトリのディレクトリ構造を視覚的に表示し、さらに統計情報も提供してくれる便利なツールです。

```typescript
import { createTool } from "@mastra/core/tools";
import { z } from "zod";
import { exec } from "child_process";
import { promisify } from "util";
import fs from "fs/promises";
import path from "path";

const execAsync = promisify(exec);

/**
 * tree構造解析ツール
 * リポジトリのディレクトリ構造を解析し、依存関係も分析します
 */
export const treeAnalyzerTool = createTool({
    id: "tree-analyzer",
    description:
        "treeコマンドを使ってリポジトリの構造を解析します。特定の拡張子でフィルタリングも可能です。",
    inputSchema: z.object({
        repositoryPath: z.string().describe("リポジトリのパス"),
        includeExtensions: z
            .array(z.string())
            .optional()
            .describe('含める拡張子のリスト (例: ["js", "ts", "json"])'),
        excludeExtensions: z
            .array(z.string())
            .optional()
            .describe("除外する拡張子のリスト"),
        maxDepth: z.number().optional().describe("表示する最大深度"),
        includeHidden: z
            .boolean()
            .optional()
            .default(false)
            .describe("隠しファイル（.で始まる）を含めるか"),
        excludePatterns: z
            .array(z.string())
            .optional()
            .describe("除外するパターン（.gitignore形式）"),
    }),
    outputSchema: z.object({
        success: z.boolean(),
        message: z.string(),
        tree: z.string().optional(),
        fileTypes: z.record(z.string(), z.number()).optional(),
        directoryCount: z.number().optional(),
        fileCount: z.number().optional(),
        treeJson: z.any().optional(),
    }),
    execute: async ({ context }) => {
        const {
            repositoryPath,
            includeExtensions,
            excludeExtensions,
            maxDepth,
            includeHidden,
            excludePatterns,
        } = context;

        try {
            // treeコマンドがインストールされているか確認
            try {
                await execAsync("which tree");
            } catch (error) {
                return {
                    success: false,
                    message:
                        'treeコマンドがインストールされていません。"brew install tree" でインストールしてください。',
                };
            }

            // .gitignoreの読み込み
            let gitignorePatterns: string[] = [];
            try {
                const gitignorePath = path.join(repositoryPath, ".gitignore");
                const gitignoreContent = await fs.readFile(
                    gitignorePath,
                    "utf-8"
                );
                gitignorePatterns = gitignoreContent
                    .split("\n")
                    .map((line) => line.trim())
                    .filter((line) => line && !line.startsWith("#"));
            } catch (error) {
                // .gitignoreが見つからない場合は無視
            }

            // .dockerignoreの読み込み
            let dockerignorePatterns: string[] = [];
            try {
                const dockerignorePath = path.join(
                    repositoryPath,
                    ".dockerignore"
                );
                const dockerignoreContent = await fs.readFile(
                    dockerignorePath,
                    "utf-8"
                );
                dockerignorePatterns = dockerignoreContent
                    .split("\n")
                    .map((line) => line.trim())
                    .filter((line) => line && !line.startsWith("#"));
            } catch (error) {
                // .dockerignoreが見つからない場合は無視
            }

            // コマンドを構築
            let command = `cd "${repositoryPath}" && tree`;

            // オプションを追加
            const options = [];

            // JSON出力
            options.push("-J");

            // 隠しファイル
            if (includeHidden) {
                options.push("-a");
            }

            // 深さ制限
            if (maxDepth !== undefined) {
                options.push(`-L ${maxDepth}`);
            }

            // ファイル拡張子フィルター
            if (includeExtensions && includeExtensions.length > 0) {
                options.push(
                    `-P "*.(${includeExtensions.join("|")})" --matchdirs`
                );
            }

            // 除外ファイル拡張子
            if (excludeExtensions && excludeExtensions.length > 0) {
                options.push(`-I "*.(${excludeExtensions.join("|")})"`);
            }

            // .gitignore & .dockerignore & カスタムパターン
            const allExcludePatterns = [
                ...gitignorePatterns,
                ...dockerignorePatterns,
                ...(excludePatterns || []),
            ];

            if (allExcludePatterns.length > 0) {
                // ワイルドカードをエスケープ
                const escapedPatterns = allExcludePatterns
                    .map((pattern) => {
                        // ディレクトリ名を抽出（末尾の/を削除）
                        const patternWithoutSlash = pattern.endsWith("/")
                            ? pattern.slice(0, -1)
                            : pattern;
                        return patternWithoutSlash;
                    })
                    .filter((pattern) => pattern && !pattern.startsWith("."))
                    .join("|");

                if (escapedPatterns) {
                    options.push(`-I "${escapedPatterns}"`);
                }
            }

            // オプションを追加
            if (options.length > 0) {
                command += ` ${options.join(" ")}`;
            }

            // コマンド実行
            const { stdout, stderr } = await execAsync(command);

            if (stderr) {
                return {
                    success: false,
                    message: `treeコマンドの実行中にエラーが発生しました: ${stderr}`,
                };
            }

            // テキスト形式のツリーを取得
            const { stdout: stdoutText } = await execAsync(
                `cd "${repositoryPath}" && tree -C --noreport`
            );

            // 結果を解析
            let treeJson;
            try {
                treeJson = JSON.parse(stdout);
            } catch (error: any) {
                return {
                    success: false,
                    message: `treeコマンドの出力をJSONとしてパースできませんでした: ${error.message}`,
                };
            }

            // ファイルタイプとディレクトリ/ファイル数を集計
            const stats = analyzeTreeJson(treeJson);

            return {
                success: true,
                message: "ディレクトリ構造の解析が完了しました。",
                tree: stdoutText,
                fileTypes: stats.fileTypes,
                directoryCount: stats.directoryCount,
                fileCount: stats.fileCount,
                treeJson,
            };
        } catch (error: any) {
            return {
                success: false,
                message: `ディレクトリ構造の解析に失敗しました: ${error.message}`,
            };
        }
    },
});

/**
 * TreeコマンドのJSON出力を解析して統計情報を生成
 */
function analyzeTreeJson(treeJson: any): {
    fileTypes: Record<string, number>;
    directoryCount: number;
    fileCount: number;
} {
    const fileTypes: Record<string, number> = {};
    let directoryCount = 0;
    let fileCount = 0;

    function traverse(node: any) {
        if (node.type === "directory") {
            directoryCount++;
            if (node.contents) {
                node.contents.forEach(traverse);
            }
        } else if (node.type === "file") {
            fileCount++;
            const nameParts = node.name.split(".");
            if (nameParts.length > 1) {
                const ext = nameParts.pop()?.toLowerCase() || "";
                fileTypes[ext] = (fileTypes[ext] || 0) + 1;
            } else {
                fileTypes["(no extension)"] =
                    (fileTypes["(no extension)"] || 0) + 1;
            }
        }
    }

    if (treeJson.length > 0) {
        traverse(treeJson[0]);
    }

    return {
        fileTypes,
        directoryCount,
        fileCount,
    };
}
```

一見すると複雑に見えるかもしれませんが、少しずつ解説していくと、どうして特定の要素が実装されているのかがわかってきますよ。では、このツールの重要な部分を詳しく見ていきましょう！

## treeAnalyzerToolの仕組みを解説 🔧

### 1. 柔軟な入力オプション 🎛️

このツールはとても柔軟な入力オプションを持っています：

```typescript
inputSchema: z.object({
    repositoryPath: z.string().describe("リポジトリのパス"),
    includeExtensions: z
        .array(z.string())
        .optional()
        .describe('含める拡張子のリスト (例: ["js", "ts", "json"])'),
    excludeExtensions: z
        .array(z.string())
        .optional()
        .describe("除外する拡張子のリスト"),
    maxDepth: z.number().optional().describe("表示する最大深度"),
    includeHidden: z
        .boolean()
        .optional()
        .default(false)
        .describe("隠しファイル（.で始まる）を含めるか"),
    excludePatterns: z
        .array(z.string())
        .optional()
        .describe("除外するパターン（.gitignore形式）"),
}),
```

これらのオプションにより、次のことが可能になります：

- **特定のファイルタイプだけを分析**: JavaScriptやTypeScriptだけを見たい？`includeExtensions: ["js", "ts"]`と指定するだけ！
- **不要なファイルを除外**: `node_modules`や`.git`ディレクトリを除外して、本当に重要なコードだけに集中
- **大きなリポジトリでも管理可能**: `maxDepth`を使って、ディレクトリの深さを制限し、情報過多を防止

### 2. .gitignoreと.dockerignoreの自動対応 🧠

優れたツールは「賢く」あるべきです。このツールは、リポジトリにある`.gitignore`と`.dockerignore`ファイルを自動的に読み込み、それらのパターンに従ってファイルをフィルタリングします：

```typescript
// .gitignoreの読み込み
let gitignorePatterns: string[] = [];
try {
    const gitignorePath = path.join(repositoryPath, ".gitignore");
    const gitignoreContent = await fs.readFile(
        gitignorePath,
        "utf-8"
    );
    gitignorePatterns = gitignoreContent
        .split("\n")
        .map((line) => line.trim())
        .filter((line) => line && !line.startsWith("#"));
} catch (error) {
    // .gitignoreが見つからない場合は無視
}

// .dockerignoreの読み込み
let dockerignorePatterns: string[] = [];
try {
    const dockerignorePath = path.join(
        repositoryPath,
        ".dockerignore"
    );
    const dockerignoreContent = await fs.readFile(
        dockerignorePath,
        "utf-8"
    );
    dockerignorePatterns = dockerignoreContent
        .split("\n")
        .map((line) => line.trim())
        .filter((line) => line && !line.startsWith("#"));
} catch (error) {
    // .dockerignoreが見つからない場合は無視
}
```

これにより、ビルドファイルやログファイル、ベンダーディレクトリなど、通常Gitでも無視されるファイルを自動的に除外できます。実際のコードに集中できますね！

### 3. treeコマンドのカスタマイズ 🛠️

Unixの`tree`コマンドは強力ですが、単独で使うと多くのオプションを手動で指定する必要があります。このツールは、与えられたオプションに基づいて最適なtreeコマンドを自動的に構築します：

```typescript
// コマンドを構築
let command = `cd "${repositoryPath}" && tree`;

// オプションを追加
const options = [];

// JSON出力
options.push("-J");

// その他のオプション...

// オプションを追加
if (options.length > 0) {
    command += ` ${options.join(" ")}`;
}
```

この部分が、単純なコマンド実行から洗練されたツールへと変わる鍵となっています！

### 4. 二重フォーマットの出力 📊

ただのツリー表示ではなく、JSON形式の構造化データとテキスト形式の視覚的なツリーの両方を提供します：

```typescript
// JSON形式の構造化データを取得
const { stdout, stderr } = await execAsync(command);

// テキスト形式の視覚的なツリーも取得
const { stdout: stdoutText } = await execAsync(
    `cd "${repositoryPath}" && tree -C --noreport`
);
```

JSON形式は機械的な処理に、テキスト形式は人間の読みやすさに貢献しています。これにより、AIと人間の両方にとって最適な出力となっています！

### 5. ファイルタイプとディレクトリ統計 📈

単なるディレクトリ構造だけでなく、リポジトリに関する豊富な統計情報も提供します：

```typescript
// ファイルタイプとディレクトリ/ファイル数を集計
const stats = analyzeTreeJson(treeJson);

return {
    success: true,
    message: "ディレクトリ構造の解析が完了しました。",
    tree: stdoutText,
    fileTypes: stats.fileTypes,
    directoryCount: stats.directoryCount,
    fileCount: stats.fileCount,
    treeJson,
};
```

これにより、「このリポジトリには何個のJavaScriptファイルがあるのか？」「ディレクトリとファイルの比率はどうなっているのか？」といった質問に即座に答えられるようになります。

### 6. 堅牢なエラーハンドリング 🛡️

ツールは様々なエラーケースに対応できるよう設計されています：

- treeコマンドが存在しない場合のチェック
- 出力のJSONパースエラーへの対応
- treeコマンド実行中のエラーのキャプチャ

```typescript
// treeコマンドがインストールされているか確認
try {
    await execAsync("which tree");
} catch (error) {
    return {
        success: false,
        message:
            'treeコマンドがインストールされていません。"brew install tree" でインストールしてください。',
    };
}
```

これにより、ツールはより堅牢に、ユーザーフレンドリーになります！

## treeAnalyzerToolの活用シーン 🌟

このツールは以下のような場面で特に役立ちます：

1. **新しいリポジトリを理解する際の第一歩**: 新しいプロジェクトの全体像を素早く把握
2. **言語やファイルタイプの分布確認**: プロジェクトがどの言語に重点を置いているかを分析
3. **ディレクトリ構造の最適化**: 過度に深いネストや、不均衡な構造を発見
4. **プロジェクトのドキュメント生成**: ディレクトリ構造をドキュメントに含めることで、新メンバーの理解を助ける

AIエージェントがこのツールを使うと、「このプロジェクトはどんな構造になっていますか？」という質問に、視覚的かつ定量的に答えることができます。これは非常に強力な能力です！

## treeAnalyzerToolの実装ポイント 📌

このツールを実装する際のポイントをいくつか挙げておきましょう：

1. **依存コマンドの確認**: treeコマンドが必要なので、実行前に存在チェックを行う
2. **柔軟なオプション設計**: できるだけ多くのユースケースに対応できるよう、豊富なオプションを用意
3. **既存の設定ファイルの活用**: .gitignoreや.dockerignoreを自動的に読み込んで活用
4. **複数出力形式の提供**: 人間向け（テキストツリー）とプログラム向け（JSON）の両方の出力を用意
5. **エラーハンドリング**: 様々なエラーシナリオを考慮し、明確なメッセージを提供

これらのポイントは、他のツールを作る際にも参考になるでしょう！

## README解析ツール (readmeAnalyzerTool) 📄

**ファイルパス**: `src/mastra/tools/analysis/readmeAnalyzer.ts`

プロジェクトを理解する際、ディレクトリ構造を把握したら次に重要なのは、そのプロジェクトの目的や使い方を知ることです。そのための最も基本的な情報源は「README」ファイルです。readmeAnalyzerToolは、READMEファイルを自動的に見つけ、その内容を構造化データとして抽出する便利なツールです。

```typescript
import { createTool } from "@mastra/core/tools";
import { z } from "zod";
import fs from "fs/promises";
import path from "path";
import { google } from "../../models";
import { Agent } from "@mastra/core";

/**
 * ミニエージェントの定義
 *
 * 注意: このファイル内で直接Agentを定義する理由：
 * 循環参照の問題を回避するため。
 *
 * 問題：
 * 1. src/mastra/agents/index.tsがreadmeAnalyzerToolをインポート
 * 2. readmeAnalyzerToolがAgentをインポートすると循環参照が発生
 * 3. 「ReferenceError: Cannot access 'readmeAnalyzerTool' before initialization」エラーが発生
 *
 * 解決策：
 * ローカルでAgentインスタンスを作成し、循環参照を断ち切る
 */
const miniAgent = new Agent({
    model: google("gemini-2.0-flash-001"),
    name: "miniAgent",
    instructions:
        "あなたはGitHubリポジトリを解析して、Cursor AIアシスタントのためのルールセット（チートシート）を生成するエージェントです。",
});

/**
 * README解析ツール
 * リポジトリのREADMEファイルを読み取り、構造化データを抽出します
 */
export const readmeAnalyzerTool = createTool({
    id: "readme-analyzer",
    description:
        "リポジトリのREADMEファイルを解析して重要な情報を構造化データとして抽出します",
    inputSchema: z.object({
        repositoryPath: z.string().describe("リポジトリのパス"),
    }),
    outputSchema: z.object({
        success: z.boolean(),
        message: z.string(),
        content: z.string().optional(),
        metadata: z
            .object({
                title: z.string().optional(),
                description: z.string().optional(),
                technologies: z.array(z.string()).optional(),
                architecture: z.string().optional(),
                installation: z.string().optional(),
                usage: z.string().optional(),
                contributing: z.string().optional(),
                license: z.string().optional(),
            })
            .optional(),
    }),
    execute: async ({ context }) => {
        const { repositoryPath } = context;

        try {
            // READMEファイルの候補リスト
            const readmeVariants = [
                "README.md",
                "Readme.md",
                "readme.md",
                "README",
                "README.markdown",
                "README.txt",
            ];

            // READMEファイルを探索
            let readmeContent = "";
            let readmePath = "";

            for (const variant of readmeVariants) {
                const filePath = path.join(repositoryPath, variant);
                try {
                    const stat = await fs.stat(filePath);
                    if (stat.isFile()) {
                        readmeContent = await fs.readFile(filePath, "utf-8");
                        readmePath = filePath;
                        break;
                    }
                } catch (error) {
                    // ファイルが存在しない場合は次のバリアントを試す
                    continue;
                }
            }

            if (!readmeContent) {
                return {
                    success: false,
                    message: "READMEファイルが見つかりませんでした。",
                };
            }

            // AIモデルを使用してメタデータを抽出
            const metadata = await extractMetadataWithAI(readmeContent);

            return {
                success: true,
                message: `READMEファイル ${readmePath} の解析が完了しました。`,
                content: readmeContent,
                metadata,
            };
        } catch (error: any) {
            return {
                success: false,
                message: `READMEの解析に失敗しました: ${error.message}`,
            };
        }
    },
});

/**
 * AIモデルを使用してREADMEファイルからメタデータを抽出する関数
 */
async function extractMetadataWithAI(content: string): Promise<{
    title: string;
    description: string;
    technologies: string[];
    architecture: string;
    installation: string;
    usage: string;
    contributing: string;
    license: string;
}> {
    // デフォルト値を設定
    const defaultMetadata = {
        title: "",
        description: "",
        technologies: [],
        architecture: "",
        installation: "",
        usage: "",
        contributing: "",
        license: "",
    };

    try {
        // AIモデル用のプロンプト
        const promptText = `
あなたはREADMEファイルから構造化されたメタデータを抽出する専門家です。
以下のREADMEファイルの内容を解析し、JSONフォーマットで以下の情報を抽出してください：

1. title: プロジェクトのタイトル（通常は最初の見出し）
2. description: プロジェクトの簡潔な説明
3. technologies: プロジェクトで使用されている技術スタックのリスト（配列形式）
4. architecture: プロジェクトのアーキテクチャや構造の説明
5. installation: インストール手順
6. usage: 使用方法
7. contributing: コントリビューションに関する情報
8. license: ライセンス情報

見つからない情報については空文字列または空配列を返してください。技術スタックは単語のリストとして抽出してください。

README内容:
${content}

JSON形式での回答のみ返してください：
`;

        const result = await miniAgent
            .generate(promptText)
            .then((res) => res.text);

        console.log(result);

        // JSONを抽出
        const jsonMatch =
            result.match(/```json\n([\s\S]*?)\n```/) ||
            result.match(/\{[\s\S]*\}/);

        if (jsonMatch) {
            // JSONを抽出してパース
            const jsonStr = jsonMatch[1] || jsonMatch[0];
            const extractedData = JSON.parse(jsonStr);

            // 必要なフィールドがあることを確認し、デフォルト値とマージ
            return {
                ...defaultMetadata,
                ...extractedData,
                // 技術スタックが文字列の場合は配列に変換
                technologies: Array.isArray(extractedData.technologies)
                    ? extractedData.technologies
                    : typeof extractedData.technologies === "string"
                      ? [extractedData.technologies]
                      : defaultMetadata.technologies,
            };
        }

        // JSONの抽出に失敗した場合
        console.warn(
            "AIモデルからJSONを抽出できませんでした。デフォルト値を使用します。"
        );
        return defaultMetadata;
    } catch (error) {
        console.error(
            "AIモデルによるメタデータ抽出中にエラーが発生しました:",
            error
        );
        return defaultMetadata;
    }
}
```

## README解析ツールの仕組みを解説 📋

### 1. 複数のREADMEフォーマットに対応 📖

READMEファイルには様々な命名規則があります。このツールは、一般的なバリエーションをすべてカバーするよう設計されています：

```typescript
// READMEファイルの候補リスト
const readmeVariants = [
    "README.md",
    "Readme.md",
    "readme.md",
    "README",
    "README.markdown",
    "README.txt",
];

// READMEファイルを探索
let readmeContent = "";
let readmePath = "";

for (const variant of readmeVariants) {
    const filePath = path.join(repositoryPath, variant);
    try {
        const stat = await fs.stat(filePath);
        if (stat.isFile()) {
            readmeContent = await fs.readFile(filePath, "utf-8");
            readmePath = filePath;
            break;
        }
    } catch (error) {
        // ファイルが存在しない場合は次のバリアントを試す
        continue;
    }
}
```

この柔軟な探索アプローチにより、異なるプロジェクト間でも一貫して動作します。READMEファイルの命名規則は統一されていないため、この機能は実際のリポジトリ解析では非常に重要です！

### 2. ミニエージェントアーキテクチャの活用 🤖

このツールの最も革新的な部分は、AIモデルを「ミニエージェント」として直接組み込んでいる点です：

```typescript
/**
 * ミニエージェントの定義
 *
 * 注意: このファイル内で直接Agentを定義する理由：
 * 循環参照の問題を回避するため。
 */
const miniAgent = new Agent({
    model: google("gemini-2.0-flash-001"),
    name: "miniAgent",
    instructions:
        "あなたはGitHubリポジトリを解析して、Cursor AIアシスタントのためのルールセット（チートシート）を生成するエージェントです。",
});
```

この設計によって、以下のメリットが得られます：

1. **スコープを限定した専用のAIエージェント**: READMEの解析に特化したエージェントにすることで、精度が高まります
2. **循環参照の回避**: ツールとエージェント間の循環参照問題を解決しています
3. **効率的な実装**: 単一ファイル内でエージェントを定義することで、依存関係を簡素化しています

これは、Mastraフレームワークの柔軟性と強力さを示す良い例です。機能ごとに特化したミニエージェントを作成することで、システム全体の品質を向上させることができます。

### 3. AIモデルによるデータ抽出アプローチ 🧠

従来の正規表現ベースの解析ではなく、AIモデルを使ってREADMEの内容を理解し、構造化データとして抽出しています：

```typescript
async function extractMetadataWithAI(content: string): Promise<{/*...*/}> {
    // ...
    const promptText = `
あなたはREADMEファイルから構造化されたメタデータを抽出する専門家です。
以下のREADMEファイルの内容を解析し、JSONフォーマットで以下の情報を抽出してください：
// ...`;

    const result = await miniAgent
        .generate(promptText)
        .then((res) => res.text);
    // ...
}
```

このアプローチには以下の利点があります：

1. **自然言語理解**: 厳密なフォーマットに依存せず、様々な表現スタイルから情報を抽出できます
2. **柔軟な解釈**: READMEファイルの構造が異なっていても適応できます
3. **言語に依存しない**: 英語、日本語、その他の言語でも機能します

正規表現では捉えられない複雑なパターンも、AIモデルならば理解できます。例えば「このプロジェクトはReactとExpressを使っています」という文から、技術スタックとして["React", "Express"]を抽出できるのです。

### 4. 堅牢なJSON抽出処理 🔍

AIモデルから返されたデータを、期待される形式に正規化する処理も含まれています：

```typescript
// 必要なフィールドがあることを確認し、デフォルト値とマージ
return {
    ...defaultMetadata,
    ...extractedData,
    // 技術スタックが文字列の場合は配列に変換
    technologies: Array.isArray(extractedData.technologies)
        ? extractedData.technologies
        : typeof extractedData.technologies === "string"
          ? [extractedData.technologies]
          : defaultMetadata.technologies,
};
```

例えば、AIモデルが`technologies`を文字列として返した場合（"React, Node.js"）、これを自動的に配列（["React", "Node.js"]）に変換します。また、デフォルト値とマージすることで、AIモデルが特定のフィールドを返さなかった場合でも一貫した構造を維持できます。

## README解析ツールの活用シーン 💡

このツールは以下のような場面で特に役立ちます：

1. **新しいリポジトリの概要把握**: プロジェクトの目的と主要機能を素早く理解
2. **技術スタックの特定**: プロジェクトで使用されている言語やフレームワークを自動検出
3. **インストール手順の抽出**: 環境構築に必要な手順を抽出して自動化
4. **使用方法の理解**: プロジェクトの基本的な使い方を把握
5. **ライセンス情報の確認**: 法的な制約を迅速に把握

AIエージェントがこのツールを使うと、「このプロジェクトは何を目的としていますか？」「どのような技術が使われていますか？」「どうやってインストールすればいいですか？」といった質問に直接答えられるようになります。

## README解析ツールの実装ポイント 📌

このツールを実装する際のポイントをいくつか挙げておきましょう：

1. **複数のファイル名バリエーションに対応**: README.md, Readme.md, README.txt など様々な命名規則をサポート
2. **ミニエージェントアーキテクチャの採用**: 特定のタスクに特化したエージェントを内包する設計
3. **AIモデルによる自然言語理解**: 正規表現よりも柔軟で高度な情報抽出が可能
4. **堅牢なJSONパースロジック**: AIモデルの様々な出力形式に対応
5. **データの正規化と前処理**: 一貫した形式でデータを返すための変換処理

特に注目すべきは、このツールが従来の静的な解析から、AIを活用した動的で理解に基づいた解析へと進化している点です。これにより、より高度で柔軟な情報抽出が可能になります。

## コード統計解析ツール (tokeiAnalyzerTool) 📊

**ファイルパス**: `src/mastra/tools/analysis/tokeiAnalyzer.ts`

プロジェクトの構造とドキュメントを理解したら、次に知りたいのはコードベース自体の特性です。tokeiAnalyzerToolは、高速なRust製コード解析ツール「tokei」を使って、リポジトリ内の言語分布やコード量を詳細に分析します。これにより、プロジェクトの規模や複雑さを定量的に把握できます！

```typescript
import { createTool } from "@mastra/core/tools";
import { z } from "zod";
import { exec } from "child_process";
import { promisify } from "util";

const execAsync = promisify(exec);

/**
 * tokei分析ツール
 * リポジトリの言語統計情報を収集し、コード複雑度や保守性も分析します
 */
export const tokeiAnalyzerTool = createTool({
    id: "tokei-analyzer",
    description: "tokeiを使ってリポジトリの言語統計とコード複雑度を分析します",
    inputSchema: z.object({
        repositoryPath: z.string().describe("リポジトリのパス"),
        format: z
            .enum(["json", "toml", "cbor", "yaml"])
            .optional()
            .default("json")
            .describe("出力フォーマット"),
        sortBy: z
            .enum(["files", "lines", "code", "comments", "blanks"])
            .optional()
            .default("code")
            .describe("ソート方法"),
    }),
    outputSchema: z.object({
        success: z.boolean(),
        message: z.string(),
        statistics: z.any().optional(),
        languageSummary: z
            .record(
                z.string(),
                z.object({
                    files: z.number(),
                    lines: z.number(),
                    code: z.number(),
                    comments: z.number(),
                    blanks: z.number(),
                    complexity: z.number().optional(),
                })
            )
            .optional(),
        totalSummary: z
            .object({
                files: z.number(),
                lines: z.number(),
                code: z.number(),
                comments: z.number(),
                blanks: z.number(),
                commentRatio: z.number(),
            })
            .optional(),
    }),
    execute: async ({ context }) => {
        const { repositoryPath, format, sortBy } = context;

        try {
            // tokeiがインストールされているか確認
            try {
                await execAsync("which tokei");
            } catch (error) {
                return {
                    success: false,
                    message:
                        'tokeiがインストールされていません。"brew install tokei" または "cargo install tokei" でインストールしてください。',
                };
            }

            // tokeiコマンドを構築
            let command = `cd "${repositoryPath}" && tokei --output ${format}`;

            if (sortBy) {
                command += ` --sort ${sortBy}`;
            }

            // tokeiコマンド実行
            const { stdout, stderr } = await execAsync(command);

            if (stderr) {
                return {
                    success: false,
                    message: `tokeiの実行中にエラーが発生しました: ${stderr}`,
                };
            }

            // 結果をパース
            let statistics;
            try {
                statistics = format === "json" ? JSON.parse(stdout) : stdout;
            } catch (error: any) {
                return {
                    success: false,
                    message: `tokeiの出力をパースできませんでした: ${error.message}`,
                };
            }

            // 言語ごとのサマリーを作成
            const languageSummary: Record<
                string,
                {
                    files: number;
                    lines: number;
                    code: number;
                    comments: number;
                    blanks: number;
                    complexity?: number;
                }
            > = {};

            let totalFiles = 0;
            let totalLines = 0;
            let totalCode = 0;
            let totalComments = 0;
            let totalBlanks = 0;

            if (format === "json") {
                Object.entries(statistics).forEach(
                    ([lang, data]: [string, any]) => {
                        if (lang !== "Total") {
                            const { blanks, code, comments, files, lines } =
                                data;

                            languageSummary[lang] = {
                                files,
                                lines,
                                code,
                                comments,
                                blanks,
                                // 複雑度の計算（仮の計算方法、実際にはもっと複雑）
                                complexity:
                                    Math.round((comments / (code || 1)) * 100) /
                                    100,
                            };

                            totalFiles += files;
                            totalLines += lines;
                            totalCode += code;
                            totalComments += comments;
                            totalBlanks += blanks;
                        }
                    }
                );
            }

            const totalSummary = {
                files: totalFiles,
                lines: totalLines,
                code: totalCode,
                comments: totalComments,
                blanks: totalBlanks,
                commentRatio:
                    Math.round((totalComments / (totalCode || 1)) * 100) / 100,
            };

            return {
                success: true,
                message: "tokeiによる言語統計分析が完了しました。",
                statistics,
                languageSummary,
                totalSummary,
            };
        } catch (error: any) {
            return {
                success: false,
                message: `tokeiの実行に失敗しました: ${error.message}`,
            };
        }
    },
});
```

このツールを使うと、「このプロジェクトは主にどの言語で書かれていますか？」「全体のコード行数はどれくらいですか？」「適切にコメントは書かれていますか？」といった質問に、具体的な数字を用いて答えられるようになります。これは、プロジェクトの概要を把握する上で非常に役立ちます！

## tokeiAnalyzerToolの仕組みを解説 📏

### 1. 柔軟な出力オプション 🔄

このツールは様々な出力形式に対応しています：

```typescript
format: z
    .enum(["json", "toml", "cbor", "yaml"])
    .optional()
    .default("json")
    .describe("出力フォーマット"),
sortBy: z
    .enum(["files", "lines", "code", "comments", "blanks"])
    .optional()
    .default("code")
    .describe("ソート方法"),
```

- JSON形式は機械処理に最適（デフォルト）
- TOML/YAMLは人間が読みやすい形式
- 「ファイル数」「コード行」など様々な基準でソート可能

ユースケースに応じて出力形式を選べるので、データの活用がとても便利です！

### 2. 言語ごとの詳細な統計情報 🔢

このツールの真髄は、言語ごとに細かく分類された統計情報です：

```typescript
const languageSummary: Record<
    string,
    {
        files: number;
        lines: number;
        code: number;
        comments: number;
        blanks: number;
        complexity?: number;
    }
> = {};
```

言語ごとに以下の情報が得られます：

- **ファイル数**: その言語のファイルがいくつあるか
- **総行数**: 全体の行数
- **コード行**: 実際のコードが書かれている行数
- **コメント行**: コメントの行数
- **空白行**: 空行の数
- **複雑度**: コメント率から算出される簡易的な複雑度指標

これにより、プロジェクトが「HTMLが多いフロントエンド中心」なのか「Javaが多いバックエンド中心」なのかが一目でわかります！

### 3. コード品質の簡易指標 📈

このツールはコード品質の簡易評価も提供します：

```typescript
// 複雑度の計算（仮の計算方法、実際にはもっと複雑）
complexity:
    Math.round((comments / (code || 1)) * 100) /
    100,
```

```typescript
commentRatio:
    Math.round((totalComments / (totalCode || 1)) * 100) / 100,
```

コメント率をベースにした単純な指標ですが、コードがどれだけドキュメント化されているかの目安になります。コメントが多すぎる場合は「コードが複雑すぎて説明が必要」とも解釈できますし、少なすぎる場合は「ドキュメント不足」と考えることもできます。

### 4. tokeiコマンドの活用 🔧

このツールはRustで書かれた超高速なコード解析ツール「tokei」を活用しています：

```typescript
// tokeiコマンドを構築
let command = `cd "${repositoryPath}" && tokei --output ${format}`;

if (sortBy) {
    command += ` --sort ${sortBy}`;
}

// tokeiコマンド実行
const { stdout, stderr } = await execAsync(command);
```

tokeiは非常に高速で、大規模なリポジトリでも数秒で解析できます。さらに300以上のプログラミング言語を認識でき、バイナリファイルも適切に処理できます。

### 5. 総合サマリーの生成 📑

各言語の統計に加えて、リポジトリ全体の概要も提供します：

```typescript
const totalSummary = {
    files: totalFiles,
    lines: totalLines,
    code: totalCode,
    comments: totalComments,
    blanks: totalBlanks,
    commentRatio:
        Math.round((totalComments / (totalCode || 1)) * 100) / 100,
};
```

このサマリーにより、「このプロジェクトは全体でどれくらいの規模か」を素早く把握できます。100,000行を超えるような大規模プロジェクトなのか、1,000行程度の小さなプロジェクトなのかが一目でわかります！

## tokeiAnalyzerToolの活用シーン 💼

このツールは以下のような場面で特に役立ちます：

1. **プロジェクトの規模把握**: 全体的なコード量と言語分布から規模を把握
2. **技術スタックの確認**: 主要言語とその割合から技術的傾向を分析
3. **コード品質の簡易評価**: コメント率や言語ごとの分布から保守性を推測
4. **リファクタリング対象の特定**: 特定の言語のコードが多すぎる部分を発見
5. **プロジェクト比較**: 異なるプロジェクト間でのコード量や構成の比較

AIエージェントがこのツールを使うと、「このプロジェクトは主にどの言語で書かれていますか？」「全体のコード行数はどれくらいですか？」「適切にコメントは書かれていますか？」といった質問に、具体的な数字を用いて答えられるようになります。

## tokeiAnalyzerToolの実装ポイント 📌

このツールを実装する際のポイントをいくつか挙げておきましょう：

1. **依存ツールの確認**: tokeiコマンドが必要なので、実行前に存在チェックを行う
2. **形式変換の処理**: JSON以外の形式にも対応できるよう設計
3. **集計ロジックの実装**: 言語ごとの統計を個別に集計し、さらに合計値も計算
4. **複雑度指標の提供**: 単純ながらもコード品質の目安となる指標を計算
5. **エラーハンドリング**: コマンド実行や結果解析における様々なエラーに対応

特に、tokeiコマンドの出力結果を構造化して扱いやすくする処理は、このツールの重要な価値です。tokei自体は素晴らしいツールですが、その結果をプログラム的に扱いやすくするラッパーとしての役割も果たしています！ 