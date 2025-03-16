---
title: "第10章：総合演習：GitHub解析エージェントの完成"
---

# GitHub解析エージェントを完成させよう！ 

## プロジェクトの全体構造

ここまでの章で開発したコンポーネントを組み合わせて、GitHub解析エージェントを完成させます。まずは、プロジェクトの全体構造を確認しましょう。

```
src
└── mastra
    ├── agents
    │   └── index.ts
    ├── index.ts
    ├── models
    │   └── index.ts
    └── tools
        ├── analysis
        │   ├── readmeAnalyzer.ts
        │   ├── tokeiAnalyzer.ts
        │   └── treeAnalyzer.ts
        ├── cheatsheet
        │   ├── index.ts
        │   └── saveCheatsheet.ts
        ├── github
        │   └── cloneRepository.ts
        ├── index.ts
        └── rag
            ├── fileProcessor.ts
            └── vectorQuery.ts
```

## ツールのエクスポート（tools/index.ts）

各ツールをアプリケーション全体で簡単に使えるようにするため、`tools/index.ts`ファイルでエクスポートしています。このファイルにより、各ツールが統一されたインターフェースで提供されます。

```typescript
// GitHubツール
export { cloneRepositoryTool } from "./github/cloneRepository";

// 分析ツール
export { readmeAnalyzerTool } from "./analysis/readmeAnalyzer";
export { tokeiAnalyzerTool } from "./analysis/tokeiAnalyzer";
export { treeAnalyzerTool } from "./analysis/treeAnalyzer";

// RAGツール
export { fileProcessorTool } from "./rag/fileProcessor";

// ベクトルツール
export { vectorQueryTool } from "./rag/vectorQuery";

// チートシートツール
export { saveCheatsheetTool } from "./cheatsheet";
```

このようにツールを整理することで、メインエージェントからすべてのツールに簡単にアクセスできるようになります。各章で作成したツールが、ここで一つのシステムとして統合されているのがわかりますね。 