---
title: "評価（Evals）"
---

# 魔法の力を測る：評価の術 🧪✨

素晴らしい魔法のAIエージェントを創り出したあなた！でも、その魔法がどれほど強力で正確なのか、客観的に知りたくありませんか？それこそが「評価（Evals）」という魔法の測定術の出番です！🔍

Mastraは「Evals」という特別な魔法の道具箱を提供しており、あなたの創った魔法使い（AIエージェント）の能力を自動的に、そして体系的に評価できます。この章では、魔法の測定方法、基本的な評価の指標、そしてオリジナルの評価術の創造方法について冒険していきましょう！

## 評価（Evals）の世界へようこそ 🌟

評価（Evals）とは、あなたの魔法使い（エージェント）が詠唱した魔法（出力）を、「賢者の判定」（モデルによる評価）、「古代の法則」（ルールベース評価）、「数の魔術」（統計的手法）など様々な方法で自動的に試験する術です。各評価は0から1の間の魔力値（スコア）を返し、この魔力値を水晶球（ログ）に記録して比較できます。

Mastraの評価魔法を使うことで、次のような謎を解き明かせます：

- 🎯 魔法使いの答えは質問に対して的を射ているか？
- 🔎 魔法の中に偏見や曲解の影響が潜んでいないか？
- 📚 手持ちの情報（コンテキスト）を正確に活用できているか？
- 🎭 言葉のトーンは一貫して安定しているか？
- 🛡️ 応答に有害な呪文が含まれていないか？

「でも難しそう...」と思ったあなた、心配無用です！評価の術は思ったより簡単に使えますよ。さあ、一緒に学んでいきましょう！

## 評価の術を実践する 📝

評価魔法を使うには、あなたのエージェントに評価の素（メトリクス）を加える必要があります。まずは、基本的な評価素を使う例を見てみましょう：

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

これで、あなたの魔法使いが話す言葉のトーンが一貫しているかを測定できるようになりました！「魔法使いが突然、丁寧語から砕けた言葉遣いに変わったりしていないか」をチェックする水晶球のようなものですね。✨

`mastra dev`という呪文を唱えると、Mastraの水晶球（ダッシュボード）で評価結果を見ることができます。魔法が可視化されるなんて素敵ですね！

## 魔法の自動試験：CI/CDパイプラインでの活用 🔄

Mastraの評価魔法は、「魔法の自動試験場」（CI/CDパイプライン）でも使えます！これにより、呪文の変更（コードの変更）があなたの魔法使いの能力に悪影響を与えていないか、自動的に確認できます。まるで、錬金術師が新しい薬の効果を自動テストするようなものです。

ESM魔法（ESMモジュール）をサポートする任意の試験の場（テストフレームワーク）- Vitest、Jest、Mochaなど - を使用できます。

以下は、Vitestという試験場を使った評価の例です：

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

試験結果をMastraの水晶球（ダッシュボード）で表示するには、試験場に特別な設定巻物（設定ファイル）を追加する必要があります：

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

## 魔法の測定具：標準評価メトリクス 📊

Mastraには多数の標準評価メトリクスが組み込まれており、これらは大きく3つの魔法の系統に分けられます。まるで、魔法学校の3つの主要学科のようですね！

### 1. 魔法の精度と信頼性の系統 🎯

- **`hallucination`**（幻覚） 🌀: 魔法使いが創作した幻の情報を見破る術
- **`faithfulness`**（忠実度） 📜: 魔法の結果が元の資料にどれだけ忠実かを測る
- **`content-similarity`**（コンテンツ類似性） 🔄: 二つの呪文の近さを比較する
- **`textual-difference`**（テキスト差異） 📊: 言葉の変化を数値化する
- **`completeness`**（完全性） ✅: 必要な魔法の要素がすべて揃っているか確認する
- **`answer-relevancy`**（回答関連性） 🎯: 応答が質問の的を射ているかを判定する

### 2. 状況理解の系統 🧠

- **`context-position`**（コンテキスト位置） 📍: 魔法文の中で重要情報がどう配置されているか
- **`context-precision`**（コンテキスト精度） 🔍: 背景情報の使い方の正確さを評価
- **`context-relevancy`**（コンテキスト関連性） 📋: 使われた背景情報の適切さを測定
- **`contextual-recall`**（コンテキスト再現性） 🔁: 背景情報からどれだけ適切に情報を引き出せたか

### 3. 魔法の品質の系統 💎

- **`tone`**（トーン） 🎭: 言葉の調子や雰囲気を分析
- **`toxicity`**（有害性） ☠️: 有害な呪文や不適切な言葉を検出
- **`bias`**（バイアス） ⚖️: 魔法の偏りを見つける
- **`prompt-alignment`**（プロンプト整合性） 📏: 指示通りに呪文を唱えているか確認
- **`summarization`**（要約） 📝: 要約の質を評価
- **`keyword-coverage`**（キーワードカバレッジ） 🔑: 重要な言葉が含まれているか確認

「わぁ、たくさんあるけど、どれを使えばいいの？」心配しないでください！最初は`answer-relevancy`（回答が質問に関連しているか）と`tone`（トーンが一貫しているか）から始めるのがおすすめです。そして徐々に他の評価も試していけば良いのです。魔法の習得に急ぎはありません！🧙‍♂️

## 独自の魔法測定器を創る 🛠️

標準の測定器では物足りない冒険者のために、オリジナルの評価メトリクスを作成する方法もあります！自分だけの魔法測定器を作るには、`Metric`という魔法の設計図を拡張して、`measure`という術を実装します。

### 基本的な測定器の作り方 🔨

以下は、魔法の呪文に特定の言葉（キーワード）が含まれているかをチェックする基本的な測定器の例です：

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
    // 空の魔法を処理
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

この測定器は、「魔法のレシピに必ず『塩』と『こしょう』という言葉が含まれているか確認する」といった使い方ができます。シンプルながら非常に実用的な魔法測定器ですね！🔍

### 魔法審判官：LLMを評価者として使う 👨‍⚖️

より高度な評価には、別の魔法使い（LLM）を審判官（ジャッジ）として使うこともできます。例えば、料理レシピの完全さを評価する例を見てみましょう。

まず、審判官が使う判定基準（プロンプト）を定義します：

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

次に、魔法審判官（ジャッジクラス）を作ります：

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

最後に、評価メトリクスの魔法の器を完成させます：

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

「うわぁ、すごく複雑そう！」と思ったかもしれませんが、実は「魔法のレシピを別の魔法使いに見てもらって、『これは完全？それとも何か足りない？』と尋ねている」だけなんです。一つ一つの部品は単純な役割を果たしています。魔法の世界は一見複雑でも、分解すれば理解できるものです！🧩

この魔法測定器をあなたの料理人エージェントに追加してみましょう：

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

### 魔法測定の実演 ✨

さあ、魔法測定器を使ってみましょう！レシピの完全性を測定する例です：

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

「素晴らしい！レシピが完璧です！」と審判が認めてくれました。あなたの魔法使いは一流の料理人と認定されたようです！👨‍🍳🌟

## 評価魔法の重要性 🌈

評価魔法があなたの冒険を助ける理由はたくさんあります：

1. **品質保証の盾** 🛡️: 魔法の質を客観的に評価し、改善点を見つけられます
2. **闇の魔法を見破る目** 👁️: バイアスや有害な内容を早期に発見できます
3. **成長の軌跡** 📈: 時間とともに魔法使いの成長を追跡し、上達を確認できます
4. **自動試験の魔術** 🔄: CI/CDに組み込み、新しい呪文（コード変更）が魔法の質を下げていないか確認できます
5. **比較の天秤** ⚖️: 異なる魔法モデルや呪文の効果を数値で比較できます

## 魔法測定の秘伝 📚

あなたの魔法評価をさらに強力にする秘伝をご紹介します：

1. **多角的な観測** 🔭: 一つの評価だけに頼らず、精度・文脈理解・品質など、複数の側面から評価しましょう。一人の審判だけでなく、多くの観点から判断するのです！

2. **専門分野の評価術を創る** 🏆: あなたの魔法使いが特殊な領域（医療・法律・料理など）で活躍するなら、その分野に特化した評価を作りましょう。専門家の目で見るのです！

3. **自動試験場に設置する** 🏗️: 評価魔法をCI/CDに組み込み、常に品質を監視しましょう。見張り番として働いてくれます！

4. **評価結果から学ぶ** 📝: 単に数値を見るだけでなく、その結果から魔法使いの指示（プロンプト）や設定を改善しましょう。失敗から学ぶのが最速の上達法です！

5. **人間の声も聞く** 👂: 自動評価だけでなく、実際のユーザーの感想も集めましょう。最終的に魔法を使うのは人間です。彼らの声こそ最も大切なのです！

## まとめ：魔法評価の達人への道 🏆

おめでとうございます！あなたはMastraの評価魔法（Evals）の秘密を解き明かしました。標準的な評価の宝石から、オリジナルの評価道具の作り方まで、魔法使い（AIエージェント）の力を正確に測定するための様々な方法を学びました。

評価は、魔法開発において品質を保証する重要な要素です。適切な評価指標を設定し、定期的に測定することで、あなたの魔法使いは常に最高のパフォーマンスを発揮し続けることができます。また、オリジナルの評価術を創造することで、特定の用途に合わせた精密な基準を定義することもできるようになりました。

「評価すること」は、単なる測定以上の意味があります。それは魔法使いを成長させ、改善し、信頼性を高めるための羅針盤なのです。

次の章では、Mastraアプリケーションの集大成と次なる冒険への道筋について解説します。さあ、魔法の探求を続けましょう！🧙‍♂️🔮✨ 