---
title: "第3章：Mastraフレームワークの基礎（index.ts）"
---

# Mastraフレームワークの基礎を学ぼう！🚀

ワークショップへようこそ！これから8つの章を通じて、GitHubリポジトリを解析するAIエージェントを一緒に作り上げていきます。まずは最初の一歩として、Mastraフレームワークの中心となる**index.ts**ファイルについて学んでいきましょう。

## Mastraプロジェクトの心臓部 - index.tsとは？💓

Mastraプロジェクトにおいて、**index.ts**ファイルは文字通り「すべての始まり」です。このファイルはプロジェクト全体の設定を担う司令塔のような役割を果たします。エージェントの登録、ワークフローの設定、ログの管理など、Mastraの重要な機能をここで初期化します。

さっそくサンプルコードを見てみましょう！

```typescript
import { Mastra } from "@mastra/core/mastra";
import { createLogger } from "@mastra/core/logger";
import { cursorRulesWorkflow } from "./workflows";
import { cursorRulesAgent } from "./agents";

export const mastra = new Mastra({
    agents: {
        cursorRulesAgent,
    },
    logger: createLogger({
        name: "GitHub Cursor Rules Agent",
        level: "info",
    }),
});
```

このコード、たった10数行ですが、驚くほど多くのことを実現しています！一つずつ見ていきましょう。

## コードの詳細解説 - 何をしているの？🔍

### 1. インポート部分 - 材料を集めよう
```typescript
import { Mastra } from "@mastra/core/mastra";
import { createLogger } from "@mastra/core/logger";
import { cursorRulesWorkflow } from "./workflows";
import { cursorRulesAgent } from "./agents";
```

最初の部分では、必要な「材料」を集めています：
- `Mastra` - フレームワークの本体
- `createLogger` - ログを記録するための機能
- `cursorRulesWorkflow` - 後で作成するワークフロー
- `cursorRulesAgent` - 後で作成するエージェント

これらのコンポーネントが、これから作るAIエージェントの基盤となります！

### 2. Mastraインスタンスの作成 - 魔法の始まり
```typescript
export const mastra = new Mastra({
    // 設定が続きます...
});
```

ここでMastraインスタンスを作成し、エクスポートしています。これにより、プロジェクトの他の部分からこのインスタンスを利用できるようになります。これがすべての機能の「入り口」となるのです！

### 3. エージェントの登録 - AIアシスタントを招待
```typescript
agents: {
    cursorRulesAgent,
},
```

ここでは、私たちが作成する`cursorRulesAgent`を登録しています。後の章で作るこのエージェントは、GitHubリポジトリを解析してCursor Rulesを生成する優れた能力を持つでしょう！

### 4. ロガーの設定 - 活動記録をつける
```typescript
logger: createLogger({
    name: "GitHub Cursor Rules Agent",
    level: "info",
}),
```

最後に、ロガーを設定しています。これはエージェントの活動記録をとるための機能です。問題が発生したときのデバッグや、エージェントの動作を追跡するのに役立ちます。
- `name` - ログに表示される名前
- `level` - 記録するログのレベル（info, warn, error, debugなど）

## 実践してみよう！💪

それでは、実際にこのファイルを作成してみましょう。プロジェクトのルートディレクトリに`src/mastra/index.ts`というファイルを作成し、上記のコードを貼り付けてください。

もちろん、今の時点では`cursorRulesAgent`はまだ作成していないため、エラーが出るかもしれません。心配いりません！これから順番に作っていきますので、今はこのファイルがどのような役割を果たすのかを理解しておくことが重要です。

## ここまでのまとめ📝

この章では、Mastraフレームワークの基礎と、特にindex.tsファイルの重要性について学びました：

- Mastraプロジェクトの中心となる`index.ts`の役割
- Mastraインスタンスの初期化方法
- エージェントの登録方法
- ロガーの設定方法

次の章では、AIモデルの設定と活用方法について学んでいきます。さまざまなAIモデル（Google Gemini、Claude、OpenAIなど）の設定方法やAPIキーの管理、モデル選択の基準について詳しく解説します。

> 📘 **完全なコードを確認したい方へ**
> 
> 本書で開発するGitHubリポジトリ解析エージェントの完全な実装は、以下のリポジトリで確認できます：
> [https://github.com/serinuntius/github-cursor-rules-agent](https://github.com/serinuntius/github-cursor-rules-agent)
> 
> このリポジトリには、本書の各章で説明するすべてのコードが実際に動作する形で含まれています。学習の参考にご活用ください。

準備はいいですか？次のステップに進みましょう！🚀 