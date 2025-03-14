---
title: "評価（Evals）"
---

# 評価（Evals）

AIエージェントの開発において、その出力品質を客観的に評価するプロセスは極めて重要です。Mastraは「Evals」（評価）と呼ばれる機能を提供し、AIエージェントの出力を自動的かつ体系的に評価することができます。この章では、Mastraでの評価方法、標準的な評価メトリクス、そしてカスタム評価の作成方法について解説します。

## 評価（Evals）の概要

評価（Evals）は、エージェントの出力を、モデルによる評価、ルールベースの評価、統計的手法など様々な方法で自動的にテストするものです。各評価は0から1の間の正規化されたスコアを返し、このスコアをログに記録して比較することができます。

Mastraの評価機能を利用することで、以下のような問いに答えることができます：

- エージェントの回答は質問に対して関連性があるか？
- 出力に偏見やバイアスが含まれていないか？
- コンテキストの情報を正確に利用しているか？
- テキストのトーンは一貫しているか？
- 回答に有害なコンテンツが含まれていないか？

## 評価の使用方法

評価を使用するには、エージェントに評価メトリクスを追加する必要があります。以下は、標準的な評価メトリクスを使用する例です：

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

この設定により、エージェントの出力のトーン一貫性を評価することができます。`mastra dev`コマンドを使用すると、Mastraダッシュボードで評価結果を確認できます。

## CI/CDパイプラインでの評価実行

Mastraの評価は、CI/CDパイプラインでも実行できます。これにより、コードの変更がエージェントの出力品質に悪影響を与えていないことを自動的に確認できます。

ESMモジュールをサポートする任意のテストフレームワーク（Vitest、Jest、Mochaなど）を使用できます。

以下は、Vitestを使用した評価の例です：

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

テスト結果をMastraダッシュボードで表示するには、テストフレームワークに特定の設定ファイルを追加する必要があります：

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

## Mastraが提供する標準評価メトリクス

Mastraには多数の標準評価メトリクスが組み込まれており、これらは大きく3つのカテゴリに分けられます。

### 1. 精度と信頼性

- **`hallucination`**（幻覚）: 創作された情報や根拠のない情報を検出します
- **`faithfulness`**（忠実度）: 出力がソース資料とどれだけ一致しているかをチェックします
- **`content-similarity`**（コンテンツ類似性）: テキストの類似性を比較します
- **`textual-difference`**（テキスト差異）: テキストの変更を測定します
- **`completeness`**（完全性）: 必要な情報がすべて含まれているかを測定します
- **`answer-relevancy`**（回答関連性）: 回答が入力質問にどれだけ対応しているかを測定します

### 2. コンテキスト理解

- **`context-position`**（コンテキスト位置）: 応答におけるコンテキストの配置を評価します
- **`context-precision`**（コンテキスト精度）: コンテキストの使用精度を評価します
- **`context-relevancy`**（コンテキスト関連性）: 使用されたコンテキストの関連性を測定します
- **`contextual-recall`**（コンテキスト再現性）: コンテキストからの情報の再現を評価します

### 3. 出力品質

- **`tone`**（トーン）: 文体やトーンを分析します
- **`toxicity`**（有害性）: 有害または不適切なコンテンツを検出します
- **`bias`**（バイアス）: 出力の潜在的なバイアスを検出します
- **`prompt-alignment`**（プロンプト整合性）: プロンプト指示への遵守度を測定します
- **`summarization`**（要約）: 要約の品質を評価します
- **`keyword-coverage`**（キーワードカバレッジ）: 重要な用語の存在を確認します

## カスタム評価の作成

標準メトリクスでは不十分な場合、独自の評価メトリクスを作成することもできます。カスタム評価を作成するには、`Metric`クラスを拡張して`measure`メソッドを実装します。

### 基本的なカスタム評価の例

以下は、出力に特定のキーワードが含まれているかをチェックする基本的なカスタム評価の例です：

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
    // 空の文字列を処理
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

### LLMを評価者として使用するカスタム評価

より高度な評価では、別のLLMをジャッジ（評価者）として使用することができます。例えば、レシピの完全性を評価する例を考えてみましょう。

まず、評価に使用するプロンプトを定義します：

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

次に、ジャッジクラスを作成します：

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

最後に、評価メトリクスクラスを作成します：

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

このカスタム評価メトリクスをエージェントに追加します：

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

### 評価の使用例

評価メトリクスを使用する例を示します：

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

## 評価の役割と重要性

評価は以下の理由でAIエージェント開発において重要な役割を果たします：

1. **品質保証**: エージェントの出力品質を客観的に評価し、改善点を特定できます
2. **バイアスと有害性の検出**: 潜在的な問題のある出力を早期に発見できます
3. **継続的改善**: 時間とともにエージェントのパフォーマンスを追跡し、改善できます
4. **自動テスト**: CI/CDパイプラインに組み込むことで、変更が品質に悪影響を与えていないことを確認できます
5. **比較評価**: 異なるモデルやプロンプト設定の効果を定量的に比較できます

## 評価のベストプラクティス

AIエージェントの評価に関するいくつかのベストプラクティスをご紹介します：

1. **複数のメトリクスを使用する**: 単一の評価メトリクスだけに頼らず、精度、コンテキスト理解、出力品質など、複数の側面から評価します
2. **ドメイン固有の評価を作成する**: 特定のユースケースや業界に合わせたカスタム評価を作成します
3. **CI/CDに統合する**: 自動テストの一部として評価を組み込み、品質の継続的なモニタリングを実現します
4. **評価結果をフィードバックに活用する**: 評価結果からエージェントのプロンプトや設定の改善点を特定します
5. **人間のフィードバックと組み合わせる**: 自動評価だけでなく、実際のユーザーフィードバックも収集し、総合的な評価を行います

## まとめ

この章では、Mastraの評価機能（Evals）について詳しく解説しました。標準的な評価メトリクスからカスタム評価の作成方法まで、AIエージェントの出力品質を客観的に評価するための様々なアプローチを学びました。

評価は、AIエージェントの開発において品質保証の重要な要素です。適切な評価メトリクスを設定し、継続的に評価することで、エージェントの出力品質を維持・向上させることができます。また、カスタム評価を作成することで、特定のユースケースに合わせた詳細な評価基準を定義することも可能です。

次の章では、Mastraアプリケーションの結論と次のステップについて解説します。 