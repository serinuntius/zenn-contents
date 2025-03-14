---
title: "実際に簡単なエージェントを作ってみよう（ワークショップ）"
---

# GitHub解析エージェント開発ワークショップへようこそ！🚀

前章で無事にMastraの環境構築とプロジェクト構造の把握ができましたね。素晴らしい！いよいよここからが本番、魔法のような体験の始まりです！この章では、あなたも簡単に作れる実用的なエージェントを一緒に開発しながら、Mastraの真の力を体感していきましょう。心配ありません、一歩一歩進めていくので、初心者の方でも大丈夫です！

今回私たちが挑戦するのは、GitHubリポジトリを解析して、なんとCursor Rulesを自動で作ってくれる便利なエージェントです。難しそう？いいえ、そんなことありません！この冒険を一緒に楽しみながら、サクッと作り上げていきましょう。あなたも驚くほどシンプルに実装できますよ！

## GitHub Cursor Rules生成エージェントの概要 ✨

まずは「このエージェントって何ができるの？」という疑問をスッキリ解消しましょう！

### エージェントの目的と機能

このエージェントは、あなたのコーディング体験を劇的に向上させる魔法の杖のような存在です：

1. GitHubのリポジトリを**ワンクリックで**自動クローン
2. リポジトリ内のコードを**賢く丁寧に**解析
3. コードの特徴や構造を**深く理解**して、CursorというAIアシスタントが使える特別なルール（.mdc）を**自動生成**

「え、Cursor Rulesって何？」と思った方も安心してください！簡単に言うと、CursorというAIコーディングアシスタントがあなたのプロジェクトをもっと深く理解するための「チートシート」のようなものです。これがあると、Cursorがそのプロジェクト特有の知識を持った状態で、あなたのコーディングをより的確にサポートしてくれるんです。すごくないですか？

### なぜこのエージェントが超便利なのか

このエージェントを作るメリットは、想像以上にワクワクするものばかりです：

1. **退屈な作業が一瞬で完了**: 手動でコードベースを解析してCursor Rulesを作るって、正直めちゃくちゃ大変ですよね。でも心配無用！このエージェントがあれば、あなたはコーヒーを一口飲む間に完了します！☕

2. **AIだからこそ見つける隠れた宝石**: 人間の目では見落としがちな細かなパターンや構造までAIが徹底分析。その結果、ダイヤモンドのように輝く高品質なCursor Rulesが生まれます。✨

3. **楽しく学べるMastraの魔法**: 「学ぶ」というより「冒険する」感覚で、Mastraの重要な機能（ツール、ワークフローなど）が自然と身につきます。気づいたら、あなたもMastra活用の達人に！🧙‍♂️

4. **今日作って、今日から使える**: 作ったエージェントは理論上の産物ではなく、あなたの明日のコーディングを即座に支援する実用的なツールになります。効率化の恩恵をすぐに実感できますよ！🛠️

さあ、エキサイティングな冒険の始まりです！一緒にこの素晴らしいエージェントを作り上げていきましょう！あなたのコーディング体験が劇的に変わる瞬間を、今から一緒に楽しみましょう！🚀

## エージェントを作る: GitHubデータ取得と解析 🔍

さあ、いよいよ本格的な実装に移ります！ワクワクしますね。ここからは魔法使いが呪文を唱えるように、GitHubリポジトリを自在に操るツールを一緒に作り上げていきましょう。難しく感じるかもしれませんが、一つ一つのステップは驚くほどシンプルなので、安心して冒険を続けてください！

### 必要なツールの作成 🛠️

私たちのエージェントには、いくつかの魔法の道具（ツール）が必要です。まるでヒーローが特殊装備を身につけるように、これからエージェントに力を与えていきましょう！Mastraの素晴らしい点は、Zodスキーマを使って入出力をキレイに型付けできることです。これによって、エージェントが「何を受け取り、何を返すのか」が一目瞭然になりますよ！

#### 1. GitHubリポジトリクローンツール ✨

まずは第一の魔法の道具、「クローンツール」を作りましょう！このツールがあれば、GitHubの星（リポジトリ）をあなたのローカル宇宙に召喚できます。以下のコードを`src/mastra/tools/github-clone-tool.ts`として保存してみましょう。コードは少し長いように見えますが、各部分は非常にシンプルな役割を持っているので心配いりません！

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

このコードを見て「うわ、複雑そう...」と思った方、安心してください！実はとてもシンプルな流れになっています。まず入力と出力のスキーマを定義して、あとは「リポジトリURLをもらったら→一時フォルダを作って→クローンして→結果を返す」という料理レシピのような明確なステップを実行しているだけなんです。素敵ですね！✨

#### 2. ファイル解析ツール 🔎

次に作るのは、「ファイル解析ツール」です。このツールは、クローンしたリポジトリを探検家のように隅々まで調査し、重要な情報を収集してくれる賢い助手です。以下のコードを`src/mastra/tools/repo-analysis-tool.ts`として保存してください。

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

このコードを見て「うわぁ、長い...」と思ったあなた、実はこのコードが行っていることは、「リポジトリの中を探検して、発見したファイルの特徴をメモする」というシンプルな仕事なんです！まるで森の中を歩いて植物や動物を記録していく自然調査員のようなものですね。このツールのおかげで、エージェントはコードの森の地図を手に入れることができます！🗺️

### Cursor Rules生成ツールの作成 ✍️

さあ、いよいよ最後の魔法の道具、「Cursor Rules生成ツール」を作成しましょう！このツールは、前のステップで集めた情報をもとに、素敵なCursor Rulesを自動で作ってくれる芸術家のような存在です。以下のコードを`src/mastra/tools/cursor-rules-generator.ts`として保存してください。

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

このコードはまるで小説家のように、収集した情報から美しいストーリー（Cursor Rules）を紡ぎ出します！「何これ難しそう...」と思うかもしれませんが、実はこのコードがやっていることは、「集めた情報を整理して、きれいな形式でファイルに書き出す」というシンプルなことなんです。まるでレポートを書くようなものですね！🖋️

### エージェントの定義 🤖

いよいよ最後のステップ、エージェント本体の定義です。これまで作ってきた3つの魔法の道具を組み合わせて、一つの強力なエージェントを生み出します！以下のコードを`src/mastra/agents/github-analyzer-agent.ts`として保存してください。ここまで来たあなたはもう熟練の魔法使いですね！✨

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

なんと簡潔なコードでしょう！これが私たちのエージェントの「脳」となる部分です。たった数行のコードですが、その中には前のステップで作成した3つの強力なツールが組み込まれています。このエージェントは、まるで優秀な探偵のように、GitHubリポジトリの秘密を解き明かし、有用なCursor Rulesを生成してくれます。素晴らしいですね！🕵️‍♀️

## エージェントの動作確認 🎮

ここまでくれば、もう成功は目前です！Mastraエージェントの実装が完了したので、実際に魔法が働くところを見てみましょう。ワクワクしますね！

### 開発サーバーの起動 🚀

プロジェクトディレクトリで以下のコマンドを実行して、魔法の世界への扉を開きましょう：

```bash
npm run dev
```

これでサーバーが起動したら、Mastraの開発環境にアクセスできます。簡単ですね！

### エージェントとの対話 💬

Mastraの開発環境で「GitHub Analyzer Agent」を選択し、以下のようなプロンプトでエージェントに魔法を託しましょう：

```
Please analyze this GitHub repository: https://github.com/facebook/react
```

そして、魔法が始まります！エージェントはリポジトリをクローンし、コードを解析して、美しいCursor Rulesを生成してくれます。まるでAIが奏でる交響曲のようです！🎵

### 生成されたCursor Rulesの確認 📝

生成されたCursor Rulesをテキストエディタで開き、その内容をじっくり眺めてみましょう。あなたが命を吹き込んだエージェントが創り出した芸術作品です！これをCursorアプリにインポートすることで、AIがプロジェクトに関する詳しい知識を持ち、あなたのコーディングをよりスマートにサポートしてくれるようになります。素晴らしい成果ですね！🏆

## まとめ 🌟

おめでとうございます！この章では、Mastraを使ったGitHub解析エージェントの開発を通して、実用的なスキルを身につけました。最初は難しそうに見えたかもしれませんが、一歩ずつ進めることで、あなたも立派なMastraエージェント開発者になりましたね！

この冒険を通じて、ツールの作成、エージェントの定義、そして実際の動作確認まで、すべてのプロセスを体験することができました。これからも、この知識を活かして、あなただけの素晴らしいエージェントを生み出していってください。

次の章では、さらに高度なテクニックを学んでいきます。Mastraの世界はまだまだ広く、探検すべき場所がたくさんあります。引き続き、一緒に楽しい冒険を続けていきましょう！🚀 