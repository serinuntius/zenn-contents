---
title: "Workflows（ワークフロー）"
---

# ワークフロー

前章ではMastraエージェントの基本構成要素や機能について学びました。これらの知識を踏まえて、今回はエージェントに具体的な処理の流れを定義する「ワークフロー」について詳しく見ていきましょう。

## ワークフローの概要

ワークフローとは、エージェントが実行すべき処理の流れを構造化して定義したものです。複雑なタスクを複数のステップに分解し、それらのステップをどのような順序や条件で実行するかを明確に指示できます。

### なぜワークフローが必要なのか

エージェントは単純なタスクであれば単体で十分対応できますが、以下のような場合にはワークフローが非常に役立ちます：

1. **複雑な処理の分解**: 大きなタスクを小さなステップに分割することで、処理が管理しやすくなります
2. **条件分岐の実現**: 状況に応じて異なる処理パスを選択できます
3. **データの受け渡し**: ステップ間でデータを整理して受け渡すことができます
4. **エラーハンドリング**: 処理中のエラーを適切に捕捉し対応できます
5. **並列処理**: 独立したタスクを同時に実行することで処理を高速化できます

### ワークフローの基本構造

Mastraのワークフローは以下の要素から構成されます：

```typescript
import { Workflow, Step } from "@mastra/core";
import { z } from "zod";

// ワークフローの定義
const myWorkflow = new Workflow({
  name: "my-workflow",
  triggerSchema: z.object({
    input: z.string(),
  }),
});

// ステップの定義
const stepOne = new Step({
  id: "stepOne",
  execute: async ({ context }) => {
    // ステップの処理ロジック
    const input = context.machineContext?.triggerData?.input;
    return { result: `処理結果: ${input}` };
  },
});

// ワークフローにステップを追加
myWorkflow.step(stepOne).commit();
```

この例では、単純なワークフローとステップを定義しています。`triggerSchema`でワークフローの入力データの型を定義し、`Step`オブジェクトでワークフローの各ステップを実装します。

## ステップの定義

ステップはワークフローの最小実行単位です。各ステップは特定のタスクを実行し、その結果を次のステップに渡します。

### ステップの基本構造

```typescript
import { Step } from "@mastra/core";
import { z } from "zod";

const processDataStep = new Step({
  id: "processData",
  description: "データを処理するステップ",
  inputSchema: z.object({
    data: z.string().describe("処理対象のデータ"),
  }),
  outputSchema: z.object({
    processedData: z.string().describe("処理後のデータ"),
  }),
  execute: async ({ context }) => {
    const data = context.input.data;
    // データ処理ロジック
    const processedData = `processed: ${data}`;
    return { processedData };
  },
});
```

各ステップには以下の要素を定義できます：

- **id**: ステップの一意の識別子
- **description**: ステップの説明（オプション）
- **inputSchema**: 入力データのスキーマ定義（オプション）
- **outputSchema**: 出力データのスキーマ定義（オプション）
- **execute**: ステップの実行ロジック

### エージェントを呼び出すステップ

ワークフロー内からエージェントを呼び出すステップを定義することもできます：

```typescript
import { Step, Agent } from "@mastra/core";
import { openai } from "@ai-sdk/openai";

// エージェントの定義
const contentAgent = new Agent({
  name: "contentAgent",
  instructions: "ブログコンテンツの作成を支援するアシスタントです",
  model: openai("gpt-4o"),
});

// エージェントを呼び出すステップ
const generateContentStep = new Step({
  id: "generateContent",
  execute: async ({ context, mastra }) => {
    const topic = context.machineContext?.triggerData?.topic;
    const agent = mastra?.agents?.contentAgent;
    
    if (!agent) {
      throw new Error("Content agent not found");
    }
    
    const result = await agent.generate(`${topic}についての短いブログ記事を作成してください`);
    
    return { 
      content: result.text 
    };
  },
});
```

このステップでは、マシンコンテキストからトピックを取得し、それをエージェントに渡してブログ記事を生成しています。

## 制御フロー

ワークフローの処理の流れをコントロールするための仕組みをいくつか見ていきましょう。

### 順次実行

最も基本的な制御フローは、ステップを順番に実行することです：

```typescript
myWorkflow.step(stepOne).then(stepTwo).then(stepThree).commit();
```

この例では、`stepOne`、`stepTwo`、`stepThree`の順番でステップが実行されます。

### 並列実行

独立したタスクを同時に実行する場合は、以下のように記述します：

```typescript
myWorkflow.step(fetchUserData).step(fetchOrderData);
```

この例では、`fetchUserData`と`fetchOrderData`が並列で実行されます。

### 条件分岐

条件に基づいて異なるステップを実行するには、`branch`メソッドを使用します：

```typescript
import { Step, Workflow } from "@mastra/core";
import { z } from "zod";

// 入力データに基づいて分岐するワークフロー
const branchingWorkflow = new Workflow({
  name: "branching-workflow",
  triggerSchema: z.object({
    userType: z.enum(["technical", "business"]),
  }),
});

// 技術系ユーザー向けステップ
const technicalStep = new Step({
  id: "technicalStep",
  execute: async () => {
    return { response: "技術的な詳細情報をお届けします" };
  },
});

// ビジネス系ユーザー向けステップ
const businessStep = new Step({
  id: "businessStep",
  execute: async () => {
    return { response: "ビジネス価値と概要情報をお届けします" };
  },
});

// 条件分岐の定義
branchingWorkflow
  .branch(({ context }) => {
    // userTypeに基づいて分岐
    return context.machineContext?.triggerData?.userType === "technical" 
      ? "technical" 
      : "business";
  })
  .when("technical", (workflow) => workflow.step(technicalStep))
  .when("business", (workflow) => workflow.step(businessStep))
  .commit();
```

この例では、ユーザータイプに基づいて異なるステップを実行する分岐を作成しています。

### 条件付きループ

条件に基づいてステップを繰り返し実行するには、`until`または`while`メソッドを使用します：

```typescript
// 値が10以上になるまでステップを繰り返す
workflow
  .step(incrementStep)
  .until(async ({ context }) => {
    // 値が10以上になったら停止
    const result = context.getStepResult(incrementStep);
    return (result?.value ?? 0) >= 10;
  }, incrementStep)
  .then(finalStep);

// または、条件式を使う場合
workflow
  .step(incrementStep)
  .until(
    { 
      ref: { step: incrementStep, path: 'value' }, 
      query: { $gte: 10 } 
    },
    incrementStep
  )
  .then(finalStep);
```

`while`メソッドは条件が真である間、ステップを繰り返し実行します：

```typescript
workflow
  .step(incrementStep)
  .while(async ({ context }) => {
    // 値が10未満の間、継続して実行
    const result = context.getStepResult(incrementStep);
    return (result?.value ?? 0) < 10;
  }, incrementStep)
  .then(finalStep);
```

## データの流れ

ワークフロー内でのデータの受け渡しは、エージェントシステムの重要な側面です。

### ステップ間でのデータ受け渡し

前のステップの出力を次のステップの入力として使用することができます：

```typescript
// データ取得ステップ
const fetchDataStep = new Step({
  id: "fetchData",
  execute: async () => {
    // データ取得処理
    return { data: "取得したデータ" };
  },
});

// データ処理ステップ
const processDataStep = new Step({
  id: "processData",
  execute: async ({ context }) => {
    // 前のステップからデータを取得
    const data = context.machineContext?.getStepPayload("fetchData")?.data;
    // データ処理
    return { processedData: `加工済み: ${data}` };
  },
});

// ワークフローにステップを追加
myWorkflow.step(fetchDataStep).then(processDataStep).commit();
```

この例では、`fetchDataStep`で取得したデータを`processDataStep`で使用しています。`getStepPayload`メソッドを使用して、前のステップの出力データを取得しています。

### 変数マッピング

より明示的なデータの受け渡しを行うには、変数マッピングを使用できます：

```typescript
// 変数マッピングを使用したデータの受け渡し
workflow
  .step(fetchDataStep)
  .then(processDataStep, {
    variables: {
      // fetchDataStepの出力dataをprocessDataStepの入力dataにマッピング
      inputData: { step: fetchDataStep, path: 'data' }
    }
  })
  .commit();
```

この方法では、ステップ間のデータの依存関係が明確になり、コードの可読性が向上します。

## 変数

ワークフロー内で変数を使用することで、異なるステップ間で情報を共有したり、計算結果を保存したりできます。

### ワークフロー変数の使用

```typescript
import { Step, Workflow } from "@mastra/core";
import { z } from "zod";

const variableWorkflow = new Workflow({
  name: "variable-workflow",
  triggerSchema: z.object({
    userName: z.string(),
  }),
});

// 変数を設定するステップ
const setVariableStep = new Step({
  id: "setVariable",
  execute: async ({ context }) => {
    const userName = context.machineContext?.triggerData?.userName;
    
    // 変数を設定
    context.machineContext?.setVariable("greeting", `こんにちは、${userName}さん！`);
    return {};
  },
});

// 変数を使用するステップ
const useVariableStep = new Step({
  id: "useVariable",
  execute: async ({ context }) => {
    // 変数を取得
    const greeting = context.machineContext?.getVariable("greeting");
    return { message: `${greeting} 今日はどうですか？` };
  },
});

// ワークフローにステップを追加
variableWorkflow.step(setVariableStep).then(useVariableStep).commit();
```

この例では、`setVariableStep`で挨拶文を変数として設定し、`useVariableStep`でその変数を使用しています。

## 一時停止と再開

長時間実行されるワークフローや、外部からの入力を待つ必要があるワークフローには、一時停止と再開の機能が役立ちます。

### ワークフローの一時停止

```typescript
import { Step, Workflow } from "@mastra/core";
import { z } from "zod";

const suspendableWorkflow = new Workflow({
  name: "suspendable-workflow",
  triggerSchema: z.object({
    initialData: z.string(),
  }),
});

// 最初のステップ
const initialStep = new Step({
  id: "initialStep",
  execute: async ({ context }) => {
    const initialData = context.machineContext?.triggerData?.initialData;
    return { processedData: `処理開始: ${initialData}` };
  },
});

// ワークフローを一時停止するステップ
const suspendStep = new Step({
  id: "suspendStep",
  execute: async ({ context, suspend }) => {
    // ワークフローを一時停止
    await suspend({
      reason: "ユーザー入力待ち",
      data: { lastState: "初期処理完了" }
    });
    
    return {};
  },
});

// 一時停止後に実行されるステップ
const resumeStep = new Step({
  id: "resumeStep",
  execute: async ({ context }) => {
    // 一時停止時のデータと再開時のデータを取得
    const suspendData = context.machineContext?.getSuspendData();
    const resumeData = context.machineContext?.getResumeData();
    
    return { 
      finalResult: `一時停止前の状態: ${suspendData?.lastState}, 再開時の入力: ${resumeData?.userInput}` 
    };
  },
});

// ワークフローにステップを追加
suspendableWorkflow
  .step(initialStep)
  .then(suspendStep)
  .then(resumeStep)
  .commit();
```

この例では、`suspendStep`でワークフローを一時停止し、ユーザーからの入力を待ちます。ワークフローが再開されると、`resumeStep`が実行され、一時停止前のデータと再開時のデータを使用して処理を続行します。

### ワークフローの再開

一時停止したワークフローを再開するには、`resume`メソッドを使用します：

```typescript
// 一時停止したワークフローを再開
await workflow.resume({
  runId: run.runId,
  stepId: 'suspendStep',
  context: {
    userInput: 'ユーザーからの入力データ'
  }
});
```

また、ワークフローの状態を監視して自動的に再開処理を行うこともできます：

```typescript
// ワークフローの監視と再開
workflow.watch(async ({ context, activePaths }) => {
  for (const path of activePaths) {
    const suspendStepStatus = context.steps?.suspendStep?.status;
    if (suspendStepStatus === 'suspended') {
      console.log("ワークフローが一時停止しました。再開します。");
      // 新しいコンテキストでワークフローを再開
      await workflow.resume({
        runId,
        stepId: 'suspendStep',
        context: { userInput: '自動生成された入力' },
      });
    }
  }
});
```

## 動的ワークフロー

実行時の状況に応じてワークフローの構造自体を変更する動的ワークフローも作成できます。

### 動的ステップの生成

```typescript
import { Step, Workflow } from "@mastra/core";
import { z } from "zod";

const dynamicWorkflow = new Workflow({
  name: "dynamic-workflow",
  triggerSchema: z.object({
    steps: z.array(z.string()),
  }),
});

// 動的にステップを生成する初期ステップ
const generateStepsStep = new Step({
  id: "generateSteps",
  execute: async ({ context }) => {
    const stepNames = context.machineContext?.triggerData?.steps || [];
    
    // 動的なステップIDリストを返す
    return { dynamicStepIds: stepNames.map(name => `dynamic-${name}`) };
  },
});

// 生成されたステップIDに基づいて実行するステップ
const executeGeneratedSteps = new Step({
  id: "executeGenerated",
  execute: async ({ context }) => {
    const dynamicStepIds = context.machineContext?.getStepPayload("generateSteps")?.dynamicStepIds || [];
    const results = {};
    
    // 各動的ステップIDに対応する処理を実行
    for (const stepId of dynamicStepIds) {
      // 実際には、ここで各ステップに応じた処理を実行
      results[stepId] = `${stepId}の処理結果`;
    }
    
    return { results };
  },
});

// ワークフローにステップを追加
dynamicWorkflow
  .step(generateStepsStep)
  .then(executeGeneratedSteps)
  .commit();
```

この例では、入力データに基づいて動的にステップIDを生成し、それらのIDに対応する処理を実行しています。

### ワークフローファクトリ

さらに高度な例として、実行時に異なるワークフローを動的に生成するワークフローファクトリも実装できます：

```typescript
// 動的ワークフロー生成ステップ
const workflowFactoryStep = new Step({
  id: 'workflowFactory',
  execute: async ({ context, mastra }) => {
    if (!mastra) {
      throw new Error('Mastra instance not available');
    }
    
    // 入力に基づいて動的なワークフローを作成
    const dynamicWorkflow = new Workflow({
      name: `dynamic-${context.workflowType}-workflow`,
      mastra,
      triggerSchema: z.object({
        input: z.string(),
      }),
    });
    
    // ワークフロータイプに応じて異なるステップを追加
    if (context.workflowType === 'simple') {
      // シンプルなワークフロー
      const simpleStep = new Step({
        id: 'simpleStep',
        execute: async ({ context }) => {
          return { result: `Simple processing: ${context.triggerData.input}` };
        },
      });
      dynamicWorkflow.step(simpleStep).commit();
    } else {
      // 複雑なワークフロー
      const step1 = new Step({
        id: 'step1',
        execute: async ({ context }) => {
          return { intermediateResult: `First processing: ${context.triggerData.input}` };
        },
      });
      const step2 = new Step({
        id: 'step2',
        execute: async ({ context }) => {
          const intermediate = context.getStepResult(step1).intermediateResult;
          return { finalResult: `Second processing: ${intermediate}` };
        },
      });
      dynamicWorkflow.step(step1).then(step2).commit();
    }
    
    // 動的ワークフローを実行
    const run = dynamicWorkflow.createRun();
    const result = await run.start({
      triggerData: { input: context.inputData },
    });
    
    return { result };
  },
});
```

この例では、入力パラメータに基づいて異なる構造のワークフローを動的に生成し、実行しています。

## 実践例：マルチエージェントワークフロー

最後に、複数のエージェントを組み合わせたワークフローの実践例を見てみましょう。この例では、コピーライターエージェントとエディターエージェントを順番に呼び出し、コンテンツを作成・編集します。

```typescript
import { Agent, Step, Workflow, Mastra } from "@mastra/core";
import { openai } from "@ai-sdk/openai";
import { z } from "zod";

// コピーライターエージェント
const copywriterAgent = new Agent({
  name: "copywriterAgent",
  instructions: "あなたはブログ記事を作成するコピーライターです。魅力的で情報価値の高い記事を書いてください。",
  model: openai("gpt-4o"),
});

// エディターエージェント
const editorAgent = new Agent({
  name: "editorAgent",
  instructions: "あなたはブログ記事を編集するエディターです。文法ミスを修正し、構造を改善し、より読みやすくしてください。",
  model: openai("gpt-4o"),
});

// ワークフロー定義
const contentWorkflow = new Workflow({
  name: "content-workflow",
  triggerSchema: z.object({
    topic: z.string(),
  }),
});

// コピーライターステップ
const copywriterStep = new Step({
  id: "copywriterStep",
  execute: async ({ context, mastra }) => {
    const topic = context.machineContext?.triggerData?.topic;
    const agent = mastra?.agents?.copywriterAgent;
    
    if (!agent) {
      throw new Error("Copywriter agent not found");
    }
    
    const result = await agent.generate(`${topic}についての800文字程度のブログ記事を作成してください`);
    
    return { draft: result.text };
  },
});

// エディターステップ
const editorStep = new Step({
  id: "editorStep",
  execute: async ({ context, mastra }) => {
    const draft = context.machineContext?.getStepPayload("copywriterStep")?.draft;
    const agent = mastra?.agents?.editorAgent;
    
    if (!agent || !draft) {
      throw new Error("Editor agent not found or draft is missing");
    }
    
    const result = await agent.generate(`以下のブログ記事を編集してください。文法ミスを修正し、より読みやすく魅力的にしてください：\n\n${draft}`);
    
    return { finalContent: result.text };
  },
});

// ワークフローにステップを追加
contentWorkflow
  .step(copywriterStep)
  .then(editorStep)
  .commit();

// Mastraインスタンスの作成
const mastra = new Mastra({
  agents: { copywriterAgent, editorAgent },
  workflows: { contentWorkflow },
});

// ワークフローの実行
async function runContentWorkflow(topic) {
  const { runId, start } = await mastra
    .getWorkflow("content-workflow")
    .createRun();
    
  const result = await start({ 
    triggerData: { topic } 
  });
  
  console.log("最終コンテンツ:", result.results.editorStep.finalContent);
  return result;
}

// 使用例
// runContentWorkflow("人工知能の最新トレンド");
```

このワークフローでは、まずコピーライターエージェントがトピックに基づいて記事の下書きを作成し、次にエディターエージェントがその下書きを編集して最終的なコンテンツを生成します。

## まとめ

この章では、Mastraにおけるワークフローの概念と実装方法について学びました：

- ワークフローの基本構造とその目的
- ステップの定義と実装方法
- 制御フローの実現（順次実行、条件分岐、ループ）
- ステップ間のデータの流れと変数の使用
- ワークフローの一時停止と再開
- 動的ワークフローの実装
- 複数エージェントを組み合わせた実践例

ワークフローを活用することで、複雑なタスクを構造化し、エージェントの能力を最大限に引き出すことができます。次の章では、RAG（Retrieval Augmented Generation）技術を用いた知識ベースの構築と活用について学んでいきましょう。
