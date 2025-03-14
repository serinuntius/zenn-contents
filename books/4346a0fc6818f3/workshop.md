---
title: "実際に簡単なエージェントを作ってみよう（ワークショップ）"
---

# GitHub解析エージェント開発ワークショップへようこそ！

前章で無事にMastraの環境構築とプロジェクト構造の把握ができましたね。いよいよここからが本番です！この章では、実際に役立つエージェントを作りながら、Mastraをより深く理解していきましょう。

今回作るのは、GitHubリポジトリを解析してCursor Rulesを自動で作ってくれるエージェントです。難しそうに感じるかもしれませんが、丁寧に一緒に進めていきますので安心してください。

## GitHub Cursor Rules生成エージェントの概要

まず、「このエージェントが何をしてくれるの？」という疑問をクリアにしましょう！

### エージェントの目的と機能

このエージェントの役割は次の通りです：

1. GitHubのリポジトリを自動でクローンする
2. リポジトリ内のコードを詳しく解析する
3. コードの特徴や構造を理解して、CursorというAIが使えるルール（.mdc）を生成する

「Cursor Rules」とは、Cursorというコーディングアシスタントがコードを書きやすくするための設定ファイルです。これがあると、Cursorがそのプロジェクト特有の知識を持った状態で的確にコードを生成したり、支援してくれます。

### なぜこのエージェントが嬉しいのか

このエージェントを作るメリットはこんな感じです：

1. **面倒な作業が一瞬で完了**: 手動でコードベースを解析してCursor Rulesを作るのって結構大変ですよね。それをエージェントが一瞬でやってくれます！

2. **AIだからこそ高品質**: 人間では気づけない細かなパターンや構造までAIが徹底分析。結果、質の高いCursor Rulesが出来上がります。

3. **Mastraを実践的に学べる**: 作りながら学ぶことで、Mastraの重要な機能（ツール、ワークフローなど）の使い方が自然と身につきます。

4. **即戦力のツールが手に入る**: 作ったエージェントは実際のプロジェクトでもすぐに活用できます。

さあ、ワクワクしながら一緒に作っていきましょう！

## エージェントを作る: GitHubデータ取得と解析

まずは、GitHubリポジトリをクローンして、そのコードを解析するための基本的なツールとエージェントを作成します。

### 必要なツールの作成

GitHubリポジトリを操作するために、いくつかのツール（関数）を作成しましょう。Mastraでは、ツールの入力と出力をZodスキーマで型付けできます。

#### 1. GitHubリポジトリクローンツール

以下のコードを`src/mastra/tools/github-clone-tool.ts`として保存してください。

```typescript
import { createTool } from "@mastra/core/tools";
import { exec } from "child_process";
import fs from "fs";
import path from "path";
import util from "util";
import { z } from "zod";

const execPromise = util.promisify(exec);

// 入力スキーマの定義
const inputSchema = z.object({
  repoUrl: z.string().describe("The GitHub repository URL to clone (e.g., https://github.com/username/repo)"),
  branch: z.string().optional().describe("Optional: The branch to clone")
});

// 出力スキーマの定義
const outputSchema = z.object({
  success: z.boolean(),
  repoPath: z.string().optional(),
  message: z.string().optional(),
  error: z.string().optional()
});

export const githubCloneTool = createTool({
  id: "githubCloneTool",
  description: "Clone a GitHub repository to the local filesystem",
  inputSchema,
  outputSchema,
  execute: async ({ context }) => {
    const { repoUrl, branch } = context;
    
    try {
      // 一時ディレクトリを作成
      const tempDir = path.join(process.cwd(), "temp_repos");
      if (!fs.existsSync(tempDir)) {
        fs.mkdirSync(tempDir, { recursive: true });
      }

      // リポジトリ名を抽出
      const repoName = repoUrl.split("/").pop()?.replace(".git", "") || "repo";
      const repoPath = path.join(tempDir, repoName);

      // 既にクローン済みかチェック
      if (fs.existsSync(repoPath)) {
        return { 
          success: true, 
          repoPath, 
          message: `Repository already exists at ${repoPath}` 
        };
      }

      // クローンコマンドを構築
      let cloneCmd = `git clone ${repoUrl} ${repoPath}`;
      if (branch) {
        cloneCmd += ` --branch ${branch} --single-branch`;
      }

      // リポジトリをクローン
      const { stdout, stderr } = await execPromise(cloneCmd);
      
      if (stderr && !stderr.includes("Cloning into")) {
        throw new Error(`Git clone error: ${stderr}`);
      }

      return { 
        success: true, 
        repoPath, 
        message: `Successfully cloned ${repoUrl} to ${repoPath}` 
      };
    } catch (error) {
      console.error("Failed to clone repository:", error);
      return { 
        success: false, 
        error: String(error) 
      };
    }
  }
});
```

#### 2. ファイル解析ツール

クローンしたリポジトリ内のファイルを解析するツールを作ります。以下のコードを`src/mastra/tools/repo-analysis-tool.ts`として保存してください。

```typescript
import { createTool } from "@mastra/core/tools";
import fs from "fs";
import path from "path";
import { promisify } from "util";
import { z } from "zod";

const readdir = promisify(fs.readdir);
const stat = promisify(fs.stat);
const readFile = promisify(fs.readFile);

// 除外するディレクトリやファイル
const EXCLUDED_DIRS = [".git", "node_modules", "dist", "build", ".cache"];
const EXCLUDED_FILES = [".DS_Store", ".gitignore"];
const MAX_FILE_SIZE = 1024 * 1024; // 1MB

// 入力スキーマの定義
const inputSchema = z.object({
  repoPath: z.string().describe("The local path to the repository"),
  fileExtensions: z.array(z.string()).optional().describe("Optional: Array of file extensions to analyze (e.g. ['.js', '.ts'])"),
  maxFilesToAnalyze: z.number().optional().describe("Optional: Maximum number of files to analyze")
});

// ファイル情報スキーマ
const fileInfoSchema = z.object({
  path: z.string(),
  extension: z.string(),
  size: z.number(),
  lineCount: z.number(),
  content: z.string()
});

// ディレクトリ構造スキーマ
const directoryStructureSchema = z.record(z.object({
  type: z.string(),
  children: z.array(z.object({
    name: z.string(),
    type: z.string(),
    extension: z.string().optional()
  })).optional()
}));

// リポジトリ情報スキーマ
const repoInfoSchema = z.object({
  name: z.string(),
  path: z.string(),
  fileCount: z.number(),
  analyzedFiles: z.array(fileInfoSchema),
  fileTypes: z.record(z.number()),
  directoryStructure: directoryStructureSchema,
  summary: z.string()
});

// 出力スキーマの定義
const outputSchema = z.object({
  success: z.boolean(),
  repoInfo: repoInfoSchema.optional(),
  error: z.string().optional()
});

export const repoAnalysisTool = createTool({
  id: "repoAnalysisTool",
  description: "Analyze files in a repository and extract code structure information",
  inputSchema,
  outputSchema,
  execute: async ({ context }) => {
    const { repoPath, fileExtensions, maxFilesToAnalyze = 100 } = context;
    
    try {
      // リポジトリの基本情報を収集
      const repoInfo = {
        name: path.basename(repoPath),
        path: repoPath,
        fileCount: 0,
        analyzedFiles: [],
        fileTypes: {},
        directoryStructure: {},
        summary: ""
      };

      // 再帰的にディレクトリを走査する関数
      const scanDirectory = async (dirPath, relativeDir = "") => {
        const entries = await readdir(dirPath, { withFileTypes: true });
        
        for (const entry of entries) {
          // 除外パスをスキップ
          if (EXCLUDED_DIRS.includes(entry.name) || EXCLUDED_FILES.includes(entry.name)) {
            continue;
          }

          const fullPath = path.join(dirPath, entry.name);
          const relativePath = path.join(relativeDir, entry.name);

          if (entry.isDirectory()) {
            // ディレクトリ情報を保存
            repoInfo.directoryStructure[relativePath] = { type: "directory", children: [] };
            // 再帰的に処理
            await scanDirectory(fullPath, relativePath);
          } else if (entry.isFile()) {
            // ファイル数が制限に達したらスキップ
            if (repoInfo.analyzedFiles.length >= maxFilesToAnalyze) {
              continue;
            }

            // 特定の拡張子のみ処理
            const ext = path.extname(entry.name).toLowerCase();
            if (fileExtensions && !fileExtensions.includes(ext)) {
              continue;
            }

            // ファイル情報を収集
            try {
              const fileStat = await stat(fullPath);
              
              // 大きすぎるファイルはスキップ
              if (fileStat.size > MAX_FILE_SIZE) {
                continue;
              }

              // ファイルタイプの統計を更新
              repoInfo.fileTypes[ext] = (repoInfo.fileTypes[ext] || 0) + 1;
              
              // ファイル内容を取得（必要に応じて）
              const content = await readFile(fullPath, 'utf8');
              const lineCount = content.split('\n').length;
              
              // ファイル情報を保存
              repoInfo.analyzedFiles.push({
                path: relativePath,
                extension: ext,
                size: fileStat.size,
                lineCount,
                content: content.slice(0, 1000) + (content.length > 1000 ? "..." : "")
              });
              
              // ディレクトリ構造に追加
              const dirName = path.dirname(relativePath);
              if (!repoInfo.directoryStructure[dirName]) {
                repoInfo.directoryStructure[dirName] = { type: "directory", children: [] };
              }
              repoInfo.directoryStructure[dirName].children.push({
                name: entry.name,
                type: "file",
                extension: ext
              });

              repoInfo.fileCount++;
            } catch (err) {
              console.error(`Error processing file ${fullPath}:`, err);
            }
          }
        }
      };

      // リポジトリ全体をスキャン
      await scanDirectory(repoPath);
      
      // 簡単な要約情報を生成
      repoInfo.summary = `Analyzed ${repoInfo.fileCount} files. Found ${Object.keys(repoInfo.fileTypes).length} different file types.`;
      
      return { 
        success: true, 
        repoInfo
      };
    } catch (error) {
      console.error("Failed to analyze repository:", error);
      return { 
        success: false, 
        error: String(error) 
      };
    }
  }
});
```

### Cursor Rules生成ツールの作成

次に、Cursor Rulesを生成するツールを作成します。以下のコードを`src/mastra/tools/cursor-rules-generator.ts`として保存してください。

```typescript
import { createTool } from "@mastra/core/tools";
import fs from "fs";
import path from "path";
import { promisify } from "util";
import { z } from "zod";

const writeFile = promisify(fs.writeFile);
const mkdir = promisify(fs.mkdir);

// リポジトリ情報スキーマの簡略化バージョン（完全なスキーマは省略）
const repoInfoSchema = z.object({
  name: z.string(),
  fileCount: z.number(),
  fileTypes: z.record(z.number()),
  directoryStructure: z.record(z.any()),
  analyzedFiles: z.array(z.any())
});

// 入力スキーマの定義
const inputSchema = z.object({
  repoInfo: repoInfoSchema.describe("Repository information from the repoAnalysisTool"),
  outputDir: z.string().optional().describe("Directory to save the generated .mdc file")
});

// 出力スキーマの定義
const outputSchema = z.object({
  success: z.boolean(),
  outputPath: z.string().optional(),
  message: z.string().optional(),
  error: z.string().optional()
});

export const cursorRulesGeneratorTool = createTool({
  id: "cursorRulesGeneratorTool",
  description: "Generate Cursor Rules (.mdc file) based on repository analysis",
  inputSchema,
  outputSchema,
  execute: async ({ context }) => {
    const { repoInfo, outputDir = "./cursor_rules" } = context;
    
    try {
      // Cursor Rules用のマークダウンテンプレート
      let mdcContent = `# ${repoInfo.name} Cursor Rules\n\n`;
      
      // リポジトリの概要セクション
      mdcContent += `## Repository Overview\n\n`;
      mdcContent += `This repository contains ${repoInfo.fileCount} files. `;
      mdcContent += `The main file types are: ${Object.entries(repoInfo.fileTypes)
        .sort((a, b) => b[1] - a[1])
        .slice(0, 5)
        .map(([ext, count]) => `${ext} (${count})`)
        .join(", ")}.\n\n`;
      
      // ディレクトリ構造セクション
      mdcContent += `## Directory Structure\n\n`;
      mdcContent += `\`\`\`\n`;
      
      // 簡略化されたディレクトリ構造を構築
      const buildDirTree = (dir = "", level = 0) => {
        const indent = "  ".repeat(level);
        let tree = "";
        
        // ルートディレクトリの場合
        if (dir === "") {
          tree += `${indent}${repoInfo.name}/\n`;
          
          // 最上位のディレクトリとファイルを取得
          const topLevelEntries = Object.keys(repoInfo.directoryStructure)
            .filter(path => !path.includes("/") || path.split("/").length === 1)
            .sort();
          
          for (const entry of topLevelEntries) {
            const structure = repoInfo.directoryStructure[entry];
            if (structure && structure.type === "directory") {
              tree += buildDirTree(entry, level + 1);
            }
          }
          
          return tree;
        }
        
        // 通常のディレクトリの場合
        tree += `${indent}${path.basename(dir)}/\n`;
        
        // このディレクトリの子要素を表示
        const dirStructure = repoInfo.directoryStructure[dir];
        if (dirStructure && dirStructure.children) {
          const sortedChildren = [...dirStructure.children].sort((a, b) => {
            // ディレクトリを先に表示
            if (a.type !== b.type) return a.type === "directory" ? -1 : 1;
            return a.name.localeCompare(b.name);
          });
          
          for (const child of sortedChildren) {
            if (child.type === "file") {
              tree += `${indent}  ${child.name}\n`;
            }
          }
        }
        
        return tree;
      };
      
      mdcContent += buildDirTree();
      mdcContent += `\`\`\`\n\n`;
      
      // コード規約とパターンセクション
      mdcContent += `## Code Conventions and Patterns\n\n`;
      mdcContent += `Based on the analysis of this repository, the following conventions and patterns are observed:\n\n`;
      
      // ファイル拡張子ごとの一般的な規約（例）
      const extensionPatterns = {};
      
      Object.keys(repoInfo.fileTypes).forEach(ext => {
        switch (ext) {
          case ".ts":
          case ".tsx":
            extensionPatterns[ext] = "TypeScript files use interfaces for type definitions and follow functional programming patterns.";
            break;
          case ".js":
          case ".jsx":
            extensionPatterns[ext] = "JavaScript files use ES6+ features like arrow functions, destructuring, and template literals.";
            break;
          case ".py":
            extensionPatterns[ext] = "Python files follow PEP 8 style guide with docstrings for functions and classes.";
            break;
          // 他の拡張子も同様に追加
        }
      });
      
      // 拡張子ごとの規約を追加
      Object.entries(extensionPatterns).forEach(([ext, pattern]) => {
        mdcContent += `- **${ext} files**: ${pattern}\n`;
      });
      
      mdcContent += `\n`;
      
      // 分析されたファイルの例セクション
      mdcContent += `## Key Files and Their Purpose\n\n`;
      
      // 重要そうなファイルを選択（単純な例として、各ディレクトリの最初のファイル）
      const keyFiles = new Set();
      
      Object.entries(repoInfo.directoryStructure).forEach(([dir, structure]) => {
        if (structure.children && structure.children.length > 0) {
          const files = structure.children.filter(child => child.type === "file");
          if (files.length > 0) {
            keyFiles.add(path.join(dir, files[0].name));
          }
        }
      });
      
      // 分析対象のファイルから実際に存在するファイルを抽出
      const analyzedKeyFiles = repoInfo.analyzedFiles
        .filter(file => keyFiles.has(file.path))
        .slice(0, 5);  // 最大5つまで
      
      analyzedKeyFiles.forEach(file => {
        mdcContent += `### ${file.path}\n\n`;
        mdcContent += `**Purpose**: This file appears to ${file.path.includes("index") ? "be an entry point" : "contain implementation details"}.\n\n`;
        
        // ファイルの内容の一部を表示
        mdcContent += `**Preview**:\n\`\`\`${file.extension.replace(".", "")}\n${
          file.content.split("\n").slice(0, 10).join("\n")
        }\n...\n\`\`\`\n\n`;
      });
      
      // ベストプラクティス
      mdcContent += `## Best Practices for This Codebase\n\n`;
      mdcContent += `When working with this repository, consider these best practices:\n\n`;
      mdcContent += `1. Follow the existing project structure for new files and features\n`;
      mdcContent += `2. Match the coding style and patterns used in similar files\n`;
      mdcContent += `3. Review the key files to understand the core architecture\n`;
      mdcContent += `4. Check for any project-specific documentation before making changes\n\n`;
      
      // 出力ディレクトリが存在しない場合は作成
      await mkdir(outputDir, { recursive: true });
      
      // .mdcファイルを保存
      const outputPath = path.join(outputDir, `${repoInfo.name}_cursor_rules.mdc`);
      await writeFile(outputPath, mdcContent);
      
      return { 
        success: true, 
        outputPath,
        message: `Successfully generated Cursor Rules at ${outputPath}`
      };
    } catch (error) {
      console.error("Failed to generate Cursor Rules:", error);
      return { 
        success: false, 
        error: String(error) 
      };
    }
  }
});
```

### エージェントの定義

最後にエージェント本体を定義します。以下のコードを`src/mastra/agents/github-analyzer-agent.ts`として保存してください。

```typescript
import { Agent } from "@mastra/core/agent";
import { openai } from "@ai-sdk/openai";
import { githubCloneTool } from "../tools/github-clone-tool";
import { repoAnalysisTool } from "../tools/repo-analysis-tool";
import { cursorRulesGeneratorTool } from "../tools/cursor-rules-generator";

export const githubAnalyzerAgent = new Agent({
  name: "GitHub Analyzer Agent",
  instructions: `You are a GitHub repository analyzer that helps users understand codebases and generate Cursor Rules.

Your tasks:
1. When given a GitHub repository URL, clone it using the githubCloneTool
2. Analyze the repository structure and code using the repoAnalysisTool
3. Summarize the repository structure, key features, and coding patterns
4. Generate Cursor Rules in .mdc format using the cursorRulesGeneratorTool

For Cursor Rules creation:
- Focus on project-specific patterns, conventions, and best practices
- Include helpful context about the codebase's organization
- Format rules as markdown with clear descriptions
- Structure information in a way that helps other developers understand the codebase quickly

When analyzing the repository:
- Look for common design patterns and architecture
- Identify naming conventions and coding standards
- Note important dependencies and how they're used
- Understand the project's structure and organization

Be thorough but concise in your analysis. Ask clarifying questions if needed.`,
  model: openai("gpt-4o"),
  tools: { 
    githubCloneTool,
    repoAnalysisTool,
    cursorRulesGeneratorTool
  },
});
```

## エージェントの動作確認

Mastraエージェントの実装が完了したので、実際に動作を確認してみましょう。

### 開発サーバーの起動

プロジェクトディレクトリで以下のコマンドを実行します：

```bash
npm run dev
```

サーバーが起動したら、Mastraの開発環境にアクセスしましょう。

### エージェントとの対話

Mastraの開発環境で「GitHub Analyzer Agent」を選択し、以下のようなプロンプトでエージェントを起動します：

```
Please analyze this GitHub repository: https://github.com/facebook/react
```

エージェントはリポジトリをクローンし、コードを解析してCursor Rulesを生成します。

### 生成されたCursor Rulesの確認

生成されたCursor Rulesをテキストエディタで開き、内容を確認してみましょう。これをCursorアプリにインポートすることで、AIがプロジェクトに関する詳しい知識を持ち、より良いアシスタントとして動作してくれます。

## まとめ

この章では、Mastraを使ったGitHub解析エージェントの開発を通して、実践的なスキルを身につけました。これからも楽しくMastraで様々なエージェントを作っていきましょう！ 