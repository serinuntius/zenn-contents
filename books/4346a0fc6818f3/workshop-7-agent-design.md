---
title: "第9章：AIエージェントの設計と実装（agents）"
---

# マスターエージェントの実装 🧠✨

ここまでで、様々な便利なツールを作ってきました。コードリポジトリをクローンするツール、ディレクトリ構造を分析するツール、READMEを解析するツール、言語統計を取得するツール、そしてRAGのためのファイル処理ツールとベクトル検索ツール、さらにはチートシートを保存するツールまで。

でも、これらのツールは別々に使うより、**一つのスマートなエージェントに組み込んで連携させた方が、もっとパワフルになる**んです！そこで登場するのが「マスターエージェント」。今回は、GitHubリポジトリを解析して開発者向けのチートシートを自動生成する魔法のようなエージェントを作っていきましょう！

## リポジトリ解析エージェントの設計 🏗️

**ファイルパス**: `src/mastra/agents/cursorRulesAgent.ts`

コードリポジトリを深く理解して開発者をサポートする - これが私たちのエージェントの使命です。このエージェントは、リポジトリの構造、主要コンポーネント、コーディング規約を自動的に理解し、AI開発アシスタントのためのガイドライン（Cursor Rules）を生成します。

特に注目すべきは、**記憶機能**を備えていること。単なる一回限りの分析ではなく、継続的な会話の中で情報を蓄積し、より深い理解に基づいたサポートを提供できるんです！

## メモリシステムの実装 💾

まずは、このエージェントの記憶の仕組みを見てみましょう：

```typescript
import { Agent } from "@mastra/core/agent";
import { Memory } from "@mastra/memory";
import { LibSQLStore } from "@mastra/core/storage/libsql";
import { LibSQLVector } from "@mastra/core/vector/libsql";
import {
    cloneRepositoryTool,
    readmeAnalyzerTool,
    tokeiAnalyzerTool,
    treeAnalyzerTool,
    fileProcessorTool,
    vectorQueryTool,
    saveCheatsheetTool,
} from "../tools";
import { openRouter } from "../models";

// メモリの設定（LibSQLをストレージとベクターデータベースに使用）
const memory = new Memory({
    storage: new LibSQLStore({
        config: {
            url: process.env.DATABASE_URL || "file:local.db",
        },
    }),
    vector: new LibSQLVector({
        connectionUrl: process.env.DATABASE_URL || "file:local.db",
    }),
    options: {
        lastMessages: 30, // 会話履歴の保持数を増やす（10→30）
        semanticRecall: {
            topK: 5, // より多くの関連メッセージを取得（3→5）
            messageRange: 3, // コンテキスト範囲を拡大（2→3）
        },
        workingMemory: {
            enabled: true, // ワーキングメモリを有効化
            template: `
# リポジトリ情報
リポジトリパス: {{repositoryPath}}
主要言語: {{mainLanguage}}

# 分析ステータス
クローン: {{cloneCompleted}}
READMEの分析: {{readmeCompleted}}
言語統計: {{tokeiCompleted}}
ディレクトリ構造: {{directoryStructureCompleted}}
重要ファイル特定: {{importantFilesCompleted}}
ファイル処理: {{fileProcessingCompleted}}

# 重要ファイル
{{importantFiles}}

# 処理済みファイル
{{processedFiles}}
            `,
        },
    },
});
```

このメモリシステムには、いくつもの魔法のような機能が詰まっています：

### 1. 永続的な記憶ストレージ 📦

```typescript
storage: new LibSQLStore({
    config: {
        url: process.env.DATABASE_URL || "file:local.db",
    },
}),
```

SQLiteベースのLibSQLを使用して、会話履歴やメタデータを永続的に保存します。これにより、エージェントを再起動しても以前の情報を忘れずに持ち続けることができます！

### 2. ベクトル記憶 🧮

```typescript
vector: new LibSQLVector({
    connectionUrl: process.env.DATABASE_URL || "file:local.db",
}),
```

会話履歴を意味的に検索できるベクトルデータベースも組み込まれています。「この質問と似た話題について、前に何か話しましたか？」といった検索が可能になります。

### 3. セマンティック検索の最適化 🔍

```typescript
semanticRecall: {
    topK: 5, // より多くの関連メッセージを取得（3→5）
    messageRange: 3, // コンテキスト範囲を拡大（2→3）
},
```

関連性の高いメッセージを5つ取得し、各メッセージの前後3つのメッセージも含めてコンテキストを把握します。これにより、会話の流れをより自然に理解できるんです！

### 4. ワーキングメモリ 🧠

```typescript
workingMemory: {
    enabled: true, // ワーキングメモリを有効化
    template: `
# リポジトリ情報
リポジトリパス: {{repositoryPath}}
主要言語: {{mainLanguage}}

# 分析ステータス
クローン: {{cloneCompleted}}
...
    `,
},
```

人間の認知プロセスに似た「ワーキングメモリ」を実装しています。これは、現在の作業に関連する重要情報を構造化された形式で保持するもので、例えばリポジトリパス、進捗状況、重要ファイルなどがテンプレートに従って記録されます。

リポジトリを分析する過程で、クローンの状態、READMEの分析状況、言語統計、ディレクトリ構造など、様々な作業の進捗状況が自動的に追跡されるんです！

## マスターエージェントの定義 🦸‍♂️

メモリシステムを設定したら、いよいよエージェント本体の定義です：

```typescript
// 単一のCursor Rules生成エージェント
export const cursorRulesAgent = new Agent({
    name: "Cursor Rules生成エージェント",
    instructions: `あなたはGitHubリポジトリを解析して、Cursor AIアシスタントのためのルールセット（チートシート）を生成するエージェントです。

以下の一連のステップでリポジトリを分析します：
1. リポジトリをクローンする - クローンしたパスは常に保存し、以降のステップで参照すること
2. READMEを読んで、プロジェクトの目的と構造を理解する
3. tokeiを使用して言語統計を収集し、リポジトリの主要言語を特定する
4. treeコマンドを使用してディレクトリ構造を分析する
5. 重要なファイルを特定し、それらをベクトルデータベースに格納する計画を立てる
6. 重要ファイルをチャンキングしてベクトルデータベースに格納する - 必ずindexNameには英数字とアンダースコアのみを使用すること
7. ベクトル検索ツールを使って関連コード片を検索する
8. 収集した情報を元にCursor Rulesチートシートを作成する

リポジトリの内容を深く理解するために、以下の点に注意してください：
- プロジェクトの主要コンポーネントと依存関係を特定する
- コーディング規約とパターンを検出する
- 設計原則とアーキテクチャを理解する
- 主要な機能と実装方法を把握する

生成するルールは以下の要素を含む必要があります：
1. プロジェクトの全体構造と設計パターン
2. 重要なクラス・関数と依存関係
3. コーディング規約と命名パターン
4. ユニークなデザインパターンと実装の特徴

ステップ間の連携を行うために、処理の結果をmetadataとして返してください。
各ステップでの判断は、前のステップで得られた情報に基づいて行ってください。
会話の流れを記憶し、一連の処理として継続してください。

インデックス名には必ず英数字とアンダースコアのみを使用してください。ハイフンや特殊文字を使うとエラーになります。
例えば "hono-index" ではなく "hono_index" を使用してください。

重要な注意点：エンベディング処理でAPIキーエラーが発生した場合でも、チャンキング処理は続行してください。その場合は、収集したファイルの内容を直接分析してチートシートを作成します。

チートシート生成に関する注意：
長いチートシートを生成する場合は、複数のセクションに分割して、各セクションを個別に生成してsave-cheatsheetツールで順番に保存してください。
これにより、トークン制限を回避して詳細なチートシートを作成できます。
最初のセクション保存時はappend=falseで、それ以降のセクションはappend=trueで追記モードを使用してください。
`,
    model: openRouter,
    tools: {
        cloneRepositoryTool,
        readmeAnalyzerTool,
        tokeiAnalyzerTool,
        treeAnalyzerTool,
        fileProcessorTool,
        vectorQueryTool,
        saveCheatsheetTool,
    },
    memory,
});
```

このエージェント定義には、いくつもの注目すべき特徴があります：

### 1. 明確な作業フロー 📋

```typescript
instructions: `あなたはGitHubリポジトリを解析して...

以下の一連のステップでリポジトリを分析します：
1. リポジトリをクローンする...
...
8. 収集した情報を元にCursor Rulesチートシートを作成する
```

エージェントが行うべき作業が8つのステップに明確に分かれています。これにより、複雑な作業が整理され、段階的に実行できるようになっています。

### 2. 詳細な分析指示 🔬

```typescript
リポジトリの内容を深く理解するために、以下の点に注意してください：
- プロジェクトの主要コンポーネントと依存関係を特定する
- コーディング規約とパターンを検出する
...
```

単なる表面的な分析ではなく、コードの深い理解を促す指示が含まれています。これにより、生成されるチートシートはより価値のあるものになります。

### 3. エラー処理の指示 🛠️

```typescript
重要な注意点：エンベディング処理でAPIキーエラーが発生した場合でも、チャンキング処理は続行してください。
```

現実的な問題（APIキーエラーなど）に対する対処法も指示されており、エージェントの堅牢性を高めています。

### 4. 出力形式の指導 📝

```typescript
チートシート生成に関する注意：
長いチートシートを生成する場合は、複数のセクションに分割して...
```

トークン制限という技術的制約を克服するための具体的な方法が指示されています。これにより、大規模なリポジトリでも詳細なチートシートを作成できます。

### 5. 統合されたツールセット 🧰

```typescript
tools: {
    cloneRepositoryTool,
    readmeAnalyzerTool,
    tokeiAnalyzerTool,
    treeAnalyzerTool,
    fileProcessorTool,
    vectorQueryTool,
    saveCheatsheetTool,
},
```

これまでの章で開発してきた全てのツールが一つのエージェントに統合されています。これにより、各ツールの機能を組み合わせた複雑なワークフローが実現可能になります。

### 6. 先進的なLLMモデルの活用 🚀

```typescript
model: openRouter,
```

OpenRouterを通じて高度なLLMモデルにアクセスし、複雑な指示を理解して実行します。モデル自体はシンプルに設定されていますが、具体的なモデル（Claude、GPT-4など）は`models.ts`で設定できます。

## エージェントの動作プロセス 🔄

このエージェントは、どのように動作するのでしょうか？実際には、Mastraフレームワークでは`agent.generate`メソッドを使用して、自然言語指示によってエージェントを操作します。エージェントが自身で適切なツールを選択し実行する流れになります：

1. **リポジトリのクローン**: 自然言語でリポジトリURLを指定し、クローンを指示します。
   ```typescript
   const cloneResponse = await cursorRulesAgent.generate(
     "https://github.com/username/repo.git をクローンしてください。この結果をリポジトリパスとして記憶してください。"
   );
   ```

2. **基本情報の収集**: クローンしたリポジトリの分析を指示します。
   ```typescript
   const readmeResponse = await cursorRulesAgent.generate(
     "クローンしたリポジトリのREADMEを分析して、プロジェクトの目的と構造についての理解を深めてください。"
   );
   
   const languageResponse = await cursorRulesAgent.generate(
     "tokeiを使って言語統計を分析し、リポジトリの主要言語を特定してください。"
   );
   
   const structureResponse = await cursorRulesAgent.generate(
     "treeコマンドを使ってリポジトリのディレクトリ構造を分析してください。最大深度は3レベルに設定してください。"
   );
   ```

3. **重要ファイルの特定**: 収集した情報を元に、重要なファイルを特定するよう指示します。
   ```typescript
   const importantFilesResponse = await cursorRulesAgent.generate(
     "これまでの分析結果に基づいて、このリポジトリの重要なファイルを特定してください。主要なコンポーネント、設定ファイル、エントリーポイントなどを優先してください。"
   );
   ```

4. **ベクトルデータベースの構築**: 特定した重要ファイルをベクトルデータベースに格納するよう指示します。
   ```typescript
   const processingResponse = await cursorRulesAgent.generate(
     "特定した重要ファイルをチャンキングして、ベクトルデータベースに格納してください。dbPathはvector_store.db、indexNameはrepo_analysisを使用してください。recursiveストラテジーを適用してください。"
   );
   ```

5. **コード検索と理解**: ベクトル検索を使って関連コード片を検索するよう指示します。
   ```typescript
   const searchResponse = await cursorRulesAgent.generate(
     "ベクトルデータベースを使って、このプロジェクトの設計パターンと主要コンポーネントに関連するコード片を検索してください。dbPathはvector_store.db、indexNameはrepo_analysisを使用してください。"
   );
   ```

6. **チートシートの生成と保存**: 収集した情報を元に、チートシートを生成して保存するよう指示します。
   ```typescript
   const cheatsheetResponse = await cursorRulesAgent.generate(
     "収集した情報を元に、このリポジトリのためのCursor Rulesチートシートを生成してください。プロジェクト構造、主要コンポーネント、デザインパターン、コーディング規約のセクションを含めてください。生成したチートシートを./cheatsheets/cursor_rules.mdに保存してください。"
   );
   ```

この方法の最大の利点は、エージェント自身がコンテキストを理解し、適切なツールを選択できることです。例えば、「このリポジトリの主要コンポーネントを分析して」という抽象的な指示も、エージェントが自律的に適切なツールを組み合わせて実行できます。

また、ワーキングメモリにより、前のステップの結果が自動的に保存され、後続のステップでそれらの情報を参照できます。例えば、リポジトリパスや主要言語の情報は後続のステップの判断材料となります。

より複雑なタスクも、単一の自然言語指示で実行できます：

```typescript
const comprehensiveAnalysis = await cursorRulesAgent.generate(`
  以下のステップでリポジトリを完全に分析してください：
  1. https://github.com/microsoft/typescript をクローンする
  2. READMEとディレクトリ構造を分析する
  3. 主要言語と重要ファイルを特定する
  4. 重要ファイルをベクトルDBに格納する
  5. TypeScriptコンパイラのアーキテクチャとデザインパターンを分析する
  6. 詳細なCursor Rulesチートシートを生成して保存する
`);
```

このように、Mastraフレームワークでは自然言語を介してエージェントとコミュニケーションし、複雑なタスクを実行していきます。エージェントには与えられたツールと詳細な指示があるため、自律的に最適な方法でタスクを遂行できるのです。

## まとめ 🎯

このマスターエージェントは、以前のチャプターで作成したすべてのツールを統合して、GitHubリポジトリを深く分析し、開発者に価値のあるチートシートを提供する強力なアシスタントです。

特に注目すべき点は：

1. **会話スレッド機能**により、複雑な作業を段階的に進められる
2. **複数ツールの統合**により、それぞれの強みを活かした分析が可能
3. **構造化された指示**により、一貫性のある品質の高い出力を生成
4. **エラー処理**により、現実的な環境での堅牢な動作を実現

Mastraフレームワークのパワーを最大限に引き出したこのエージェントは、開発者の生産性を大幅に向上させ、大規模なコードベースへの参入障壁を低下させるでしょう。

これで、AIエージェントがソフトウェア開発を支援する基本的な仕組みの一例を見てきました。次のステップでは、これをさらに拡張して、より複雑な開発タスクを自動化したり、複数のエージェントが協力するマルチエージェントシステムを構築したりできます。

AIとのコラボレーションによる開発の新時代が、今、始まろうとしています！ 
