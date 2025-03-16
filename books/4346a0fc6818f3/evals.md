---
title: "第16章：エージェントの評価と改善 (執筆中 🖋️)"
---

# AIエージェントを客観的に評価しよう！ 🧪📊

素晴らしいAIエージェントを作り上げたあなた！でも、そのエージェントがどれほど効果的で正確なのか、客観的に知りたくありませんか？それこそが「評価（Evals）」という重要なプロセスの出番です！🔍

Mastraは「Evals」という強力な機能を提供しており、あなたの作ったAIエージェントの能力を自動的に、そして体系的に評価できます。この章では、エージェントの評価方法、基本的な評価指標、そしてオリジナルの評価メトリクスの作成方法について詳しく解説していきます！

## 評価（Evals）システムの重要性 🌟

評価（Evals）とは、あなたのエージェントが生成した出力を、「モデルによる評価」、「ルールベースの評価」、「統計的手法」など様々な方法で自動的に検証するシステムです。各評価は0から1の間のスコアを返し、このスコアをログに記録して比較できます。簡単に言えば、あなたのエージェントの性能を数値化するための仕組みなんです！

Mastraの評価システムを使うことで、次のような重要な問いに答えられます：

- 🎯 エージェントの回答は質問に対して適切か？
- 🔎 出力に偏見や誤った情報が含まれていないか？
- 📚 与えられたコンテキスト情報を正確に活用できているか？
- 🎭 言葉のトーンは一貫して安定しているか？
- 🛡️ 応答に有害なコンテンツが含まれていないか？

「でも難しそう...」と思ったあなた、心配無用です！評価システムは思ったより簡単に使えますよ。さあ、一緒に学んでいきましょう！

## 評価システムを実践する 📝

評価機能を使うには、あなたのエージェントに評価メトリクスを追加する必要があります。まずは、基本的な評価メトリクスを使う例を見てみましょう：

```typescript
import { Agent } from "@mastra/core/agent";
import { openai } from "@ai-sdk/openai";
import { ToneConsistencyMetric } from "@mastra/evals/nlp";

export const myAgent = new Agent({
  name: "My Agent",
  instructions: "あなたは役立つアシスタントです。",
  model: openai("gpt-4o-mini"),
  evals: {
    tone: new ToneConsistencyMetric()
  },
});
```

これで、あなたのエージェントが話す言葉のトーンが一貫しているかを測定できるようになりました！「エージェントが突然、丁寧語から砕けた言葉遣いに変わったりしていないか」をチェックする仕組みですね。✨

`mastra dev`コマンドを実行すると、Mastraのダッシュボードで評価結果をリアルタイムに確認できます。エージェントの性能が視覚化されるなんて便利ですね！

## 自動評価：CI/CDパイプラインでの活用 🔄

Mastraの評価システムは、CI/CD（継続的インテグレーション/継続的デリバリー）パイプラインでも活用できます！これにより、コードの変更があなたのエージェントの性能に悪影響を与えていないか、自動的に確認できます。まるで、研究者が新しい手法の効果を自動テストするようなものです。

ESMモジュールをサポートする任意のテストフレームワーク - Vitest、Jest、Mochaなど - を使用できます。

以下は、Vitestというテストフレームワークを使った評価の例です：

```typescript
import { describe, it, expect } from 'vitest';
import { evaluate } from '@mastra/core/eval';
import { myAgent } from './index';
import { ToneConsistencyMetric } from "@mastra/evals/nlp";

describe('My Agent', () => {
  it('トーン一貫性を検証できるはずです', async () => {
    const metric = new ToneConsistencyMetric();
    const result = await evaluate(myAgent, 'こんにちは、世界！', metric)
    expect(result.score).toBe(1);
  });
});
```

「え、これだけ？もっと複雑なのかと思った！」そうなんです、意外と簡単でしょう？😉

テスト結果をMastraのダッシュボードで表示するには、テストフレームワークに特別な設定ファイルを追加する必要があります：

```typescript
// globalSetup.ts
import { globalSetup } from '@mastra/evals';

export default function setup() {
  globalSetup()
}

// testSetup.ts
import { beforeAll } from 'vitest';
import { attachListeners } from '@mastra/evals';

beforeAll(async () => {
  await attachListeners();
});

// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globalSetup: './globalSetup.ts',
    setupFiles: ['./testSetup.ts'],
  },
})
```

## 標準評価メトリクス一覧 📊

Mastraには多数の標準評価メトリクスが組み込まれており、これらは大きく3つのカテゴリに分けられます。エージェント評価の基本となる重要な指標群です！

### 1. 精度と信頼性のメトリクス 🎯

- **`hallucination`**（幻覚） 🌀: エージェントが創作した誤情報を検出
- **`faithfulness`**（忠実度） 📜: 出力結果が元の資料にどれだけ忠実かを測定
- **`content-similarity`**（コンテンツ類似性） 🔄: 二つのテキストの近さを比較
- **`textual-difference`**（テキスト差異） 📊: テキストの変化を数値化
- **`completeness`**（完全性） ✅: 必要な要素がすべて揃っているか確認
- **`answer-relevancy`**（回答関連性） 🎯: 応答が質問に適切に答えているかを判定

### 2. コンテキスト理解のメトリクス 🧠

- **`context-position`**（コンテキスト位置） 📍: テキスト内で重要情報がどう配置されているか
- **`context-precision`**（コンテキスト精度） 🔍: 背景情報の使い方の正確さを評価
- **`context-relevancy`**（コンテキスト関連性） 📋: 使われた背景情報の適切さを測定
- **`contextual-recall`**（コンテキスト再現性） 🔁: 背景情報からどれだけ適切に情報を引き出せたか

### 3. 出力品質のメトリクス 💎

- **`tone`**（トーン） 🎭: 言葉の調子や雰囲気を分析
- **`toxicity`**（有害性） ☠️: 有害なコンテンツや不適切な言葉を検出
- **`bias`**（バイアス） ⚖️: 出力の偏りを検出
- **`prompt-alignment`**（プロンプト整合性） 📏: 指示通りに動作しているか確認
- **`summarization`**（要約） 📝: 要約の質を評価
- **`keyword-coverage`**（キーワードカバレッジ） 🔑: 重要な用語が含まれているか確認

「わぁ、たくさんあるけど、どれを使えばいいの？」心配しないでください！最初は`answer-relevancy`（回答が質問に関連しているか）と`tone`（トーンが一貫しているか）から始めるのがおすすめです。そして徐々に他の評価も試していけば良いのです。焦る必要はありません！🚀

## カスタム評価メトリクスの作成方法 🛠️

標準のメトリクスでは物足りないという方のために、オリジナルの評価メトリクスを作成する方法もあります！独自の評価指標を作るには、`Metric`というベースクラスを拡張して、`measure`メソッドを実装します。

### 基本的な評価メトリクスの作り方 🔨

以下は、テキストに特定の言葉（キーワード）が含まれているかをチェックする基本的なメトリクスの例です：

```typescript
import { Metric, type MetricResult } from '@mastra/core/eval';

interface KeywordCoverageResult extends MetricResult {
  info: {
    totalKeywords: number;
    matchedKeywords: number;
  };
}

export class KeywordCoverageMetric extends Metric {
  private referenceKeywords: Set<string>;
  
  constructor(keywords: string[]) {
    super();
    this.referenceKeywords = new Set(keywords);
  }
  
  async measure(input: string, output: string): Promise<KeywordCoverageResult> {
    // 空の入力を処理
    if (!input && !output) {
      return {
        score: 1,
        info: {
          totalKeywords: 0,
          matchedKeywords: 0,
        },
      };
    }
    
    const matchedKeywords = [...this.referenceKeywords].filter(k => output.includes(k));
    const totalKeywords = this.referenceKeywords.size;
    const coverage = totalKeywords > 0 ? matchedKeywords.length / totalKeywords : 0;
    
    return {
      score: coverage,
      info: {
        totalKeywords: this.referenceKeywords.size,
        matchedKeywords: matchedKeywords.length,
      },
    };
  }
}
```

このメトリクスは、「レシピに必ず『塩』と『こしょう』という言葉が含まれているか確認する」といった使い方ができます。シンプルながら非常に実用的な評価指標ですね！🔍

### LLMを評価者として使用する 👨‍⚖️

より高度な評価には、別のLLMを評価者（ジャッジ）として使うこともできます。例えば、料理レシピの完全性を評価する例を見てみましょう。

まず、評価者が使う判定基準（プロンプト）を定義します：

```typescript
// src/mastra/evals/recipe-completeness/prompts.ts
export const RECIPE_COMPLETENESS_INSTRUCTIONS = `あなたはレシピの完全性を評価するマスターシェフです。あなたの役割は、レシピに成功料理に必要なすべての重要な情報が含まれていることを確認することです。`;

export const generateCompletenessPrompt = ({
  input,
  output,
}: {
  input: string;
  output: string;
}) => `このレシピに成功料理に必要な重要な要素がすべて含まれているかを分析してください。

確認すべき重要な要素：
1. 材料の量
   - すべての材料に具体的な測定値が必要
   - 不完全な例: "トマトを追加"
   - 完全な例: "角切りトマト2個を追加"
2. 調理時間
   - 各調理ステップに具体的な時間が必要
   - 不完全な例: "できるまで調理"
   - 完全な例: "15-20分間調理"
3. 調理温度
   - すべての加熱ステップに具体的な温度が必要
   - 不完全な例: "オーブンで焼く"
   - 完全な例: "350°F (175°C)のオーブンで焼く"

ユーザー入力: ${input}
分析するレシピ: ${output}

以下の形式で回答してください:
{ "missing": ["不足している要素を列挙"], "verdict": "完全 または 不完全" }`;
```

次に、評価者（ジャッジクラス）を作成します：

```typescript
// src/mastra/evals/recipe-completeness/metricJudge.ts
import { type LanguageModel } from '@mastra/core/llm';
import { MastraAgentJudge } from '@mastra/evals/judge';
import { z } from 'zod';
import { RECIPE_COMPLETENESS_INSTRUCTIONS, generateCompletenessPrompt } from './prompts';

export class RecipeCompletenessJudge extends MastraAgentJudge {
  constructor(model: LanguageModel) {
    super('Recipe Completeness', RECIPE_COMPLETENESS_INSTRUCTIONS, model);
  }

  async evaluate(
    input: string,
    output: string,
  ): Promise<{ missing: string[]; verdict: string; }> {
    const completenessPrompt = generateCompletenessPrompt({ input, output });
    const result = await this.agent.generate(completenessPrompt, {
      output: z.object({
        missing: z.array(z.string()),
        verdict: z.string(),
      }),
    });
    return result.object;
  }
}
```

最後に、評価メトリクスクラスを完成させます：

```typescript
// src/mastra/evals/recipe-completeness/index.ts
import { Metric, type MetricResult } from '@mastra/core/eval';
import { type LanguageModel } from '@mastra/core/llm';
import { RecipeCompletenessJudge } from './metricJudge';

export interface MetricResultWithInfo extends MetricResult {
  info: {
    missing: string[];
    verdict: string;
  };
}

export class RecipeCompletenessMetric extends Metric {
  private judge: RecipeCompletenessJudge;
  
  constructor(model: LanguageModel) {
    super();
    this.judge = new RecipeCompletenessJudge(model);
  }
  
  async measure(input: string, output: string): Promise<MetricResultWithInfo> {
    const { verdict, missing } = await this.judge.evaluate(input, output);
    const score = verdict.toLowerCase() === 'incomplete' ? 0 : 1;
    
    return {
      score,
      info: {
        missing,
        verdict,
      },
    };
  }
}
```

「うわぁ、すごく複雑そう！」と思ったかもしれませんが、実は「レシピを別のAIモデルに見てもらって、『これは完全？それとも何か足りない？』と尋ねている」だけなんです。一つ一つの部品は単純な役割を果たしています。複雑に見えても、分解すれば理解できるものです！🧩

この評価メトリクスをあなたの料理人エージェントに追加してみましょう：

```typescript
import { openai } from '@ai-sdk/openai';
import { Agent } from '@mastra/core/agent';
import { RecipeCompletenessMetric } from '../evals';

export const chefAgent = new Agent({
  name: 'chef-agent',
  instructions: 'あなたはミシェル、実用的で経験豊富な家庭料理人です。' +
                '手元にある材料を使って料理する手助けをします。',
  model: openai('gpt-4o-mini'),
  evals: {
    recipeCompleteness: new RecipeCompletenessMetric(openai('gpt-4o-mini')),
  },
});
```

### 評価メトリクスの実際の使用例 ✨

さあ、評価メトリクスを実際に使ってみましょう！レシピの完全性を測定する例です：

```typescript
import { mastra } from './mastra';

const chefAgent = mastra.getAgent('chefAgent');
const metric = chefAgent.evals.recipeCompleteness;

// レシピを評価する例
const input = '正確な分量とタイミングのあるシンプルなパスタレシピを教えてください。';
const response = await chefAgent.generate(input);
const result = await metric.measure(input, response.text);

console.log('評価結果:', {
  score: result.score,
  missing: result.info.missing,
  verdict: result.info.verdict,
});

// 出力例:
// 評価結果: { score: 1, missing: [], verdict: '完全' }
```

「素晴らしい！レシピが完璧です！」と評価システムが判定しました。あなたのエージェントは一流の料理人の水準に達したようです！👨‍🍳🌟

## 評価システムの重要性 🌈

評価システムがあなたの開発を助ける理由はたくさんあります：

1. **品質保証** 🛡️: エージェントの出力品質を客観的に評価し、改善点を見つけられます
2. **問題の早期発見** 👁️: バイアスや有害な内容を早期に検出できます
3. **成長の追跡** 📈: 時間とともにエージェントの性能向上を追跡し、進捗を確認できます
4. **自動テスト** 🔄: CI/CDに組み込み、新しいコード変更がエージェントの品質を下げていないか確認できます
5. **比較分析** ⚖️: 異なるモデルやプロンプトの効果を数値で比較できます

## 評価システム活用のベストプラクティス 📚

あなたの評価システムをさらに効果的にするためのヒントをご紹介します：

1. **多角的な評価** 🔭: 一つの評価だけに頼らず、精度・文脈理解・品質など、複数の側面から評価しましょう。多くの観点から総合的に判断するのです！

2. **専門分野向けの評価メトリクスを開発** 🏆: あなたのエージェントが特殊な領域（医療・法律・料理など）で活躍するなら、その分野に特化した評価を作りましょう。専門家の視点で見ることが重要です！

3. **CIパイプラインに組み込む** 🏗️: 評価システムをCI/CDに組み込み、常に品質を監視しましょう。品質管理の自動化が効率的です！

4. **評価結果からの学習** 📝: 単に数値を見るだけでなく、その結果からエージェントのプロンプトや設定を改善しましょう。失敗から学ぶことが最速の成長につながります！

5. **ユーザーフィードバックも重視** 👂: 自動評価だけでなく、実際のユーザーの感想も集めましょう。最終的にエージェントを使うのは人間です。彼らの声こそ最も貴重な情報源です！

## まとめ：評価システムの達人への道 🏆

おめでとうございます！あなたはMastraの評価システム（Evals）の秘密を解き明かしました。標準的な評価メトリクスから、オリジナルの評価ツールの作り方まで、AIエージェントの性能を正確に測定するための様々な方法を学びました。

評価は、AIエージェント開発において品質を保証する重要な要素です。適切な評価指標を設定し、定期的に測定することで、あなたのエージェントは常に最高のパフォーマンスを発揮し続けることができます。また、オリジナルの評価メトリクスを開発することで、特定の用途に合わせた精密な基準を定義することもできるようになりました。

「評価すること」は、単なる測定以上の意味があります。それはエージェントを成長させ、改善し、信頼性を高めるための羅針盤なのです。

次の章では、Mastraアプリケーションの集大成と次なる展開について解説します。さあ、エージェント開発の旅を続けましょう！🚀📊✨ 