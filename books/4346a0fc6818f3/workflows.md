---
title: "Workflows（ワークフロー）"
---

# ワークフローの魔法を極めよう！ 🧙‍♂️✨

前章ではMastraエージェントの基本構成要素や機能という、魔法の材料について学びましたね！素晴らしい進歩です。これらの知識を武器に、今回はさらに一歩進んで、エージェントに具体的な魔法の詠唱手順を教える「ワークフロー」について探検していきましょう。まるで魔法使いが呪文の詠唱順序を決めるように、エージェントの行動パターンを設計できるようになりますよ！

## ワークフローの概要 - 魔法の設計図 📜

ワークフローとは、エージェントが実行すべき処理の流れを構造化して定義した、いわば「魔法の地図」のようなものです。冒険で言えば、「まずドラゴンの洞窟に行って、次に宝箱を見つけて、最後に村に戻る」といった具体的な行程表のようなもの。複雑な冒険を小さなミッションに分解し、それらをどのような順序や条件で実行するかを明確に指示できるんです！

### なぜワークフローが必要なのか 🤔

エージェントは単純な魔法（タスク）であれば単体で十分対応できますが、以下のような場合には、ワークフローという魔法の設計図が非常に役立ちます：

1. **複雑な魔法の分解** 🧩: 大きな呪文を小さな詠唱ステップに分割することで、魔法が管理しやすくなります
2. **魔法の分岐点の実現** 🌳: 「もし〇〇なら□□の魔法、そうでなければ△△の魔法」という状況判断ができます
3. **魔力の受け渡し** 💫: ステップ間で魔力（データ）をスムーズに受け渡せます
4. **魔法の失敗対策** 🛡️: 呪文詠唱中の事故を適切に捕捉し対応できます
5. **同時詠唱** ⚡: 独立した魔法を同時に詠唱することで効率アップ！

冒険で例えるなら、「雨が降ったら傘を持って行く」「敵が多いなら仲間を呼ぶ」などの状況に応じた計画が立てられるということです。賢い冒険者になるための必須スキルですね！

### ワークフローの基本構造 - 魔法の骨組み 🧠

Mastraのワークフローは以下の要素から構成されます。ちょっと見てみましょう！

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

このコードは少し複雑に見えるかもしれませんが、心配いりません！基本的には「ワークフロー」という冒険の大まかな計画を立て、その中に「ステップ」という個別のミッションを設定しているだけなんです。`triggerSchema`はこの冒険に必要な装備品リスト、`Step`オブジェクトは冒険の各立ち寄りポイントで何をするかを記した指示書のようなものですね！✨

## ステップの定義 - 魔法のレシピを書く 📝

ステップはワークフローの最小実行単位、言わば魔法のレシピの1ステップです。「卵を割る」「小麦粉を入れる」のような、魔法のレシピの各手順をステップとして定義していきます。

### ステップの基本構造 - レシピの書き方 🍳

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

「うわ、難しそう...」と思いましたか？大丈夫、実はとってもシンプルなんです！料理レシピに例えると：

- **id**: レシピの名前（例：「ふわふわオムレツ」）
- **description**: レシピの説明（「朝食にぴったりの簡単レシピ」）
- **inputSchema**: 必要な食材リスト（卵、塩、バター...）
- **outputSchema**: 完成品の仕様（ふわふわ、黄金色、半熟...）
- **execute**: 実際の調理手順（卵を割って、塩を入れて、かき混ぜて...）

このように考えると、プログラミングも料理も似ていて、ステップバイステップで美味しい結果を生み出すんですね！🍽️

### エージェントを呼び出すステップ - 魔法使いに協力を求める 🧚‍♀️

ワークフロー内から別の魔法使い（エージェント）を呼び出すステップを定義することもできます。まるで冒険中に専門家の助けを借りるようなものですね：

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

このステップでは、「私には文章を書くのが苦手だから、文章の得意な友達に手伝ってもらおう！」というようなことをやっています。冒険の途中で賢者に相談するようなものですね。ワークフローの中で専門家（エージェント）の助けを借りることで、より素晴らしい結果が得られます！🌟

## 制御フロー - 魔法の道筋を設計する 🗺️

ワークフローの処理の流れをコントロールするための仕組み、つまり魔法の道筋をどう設計するかについて見ていきましょう。冒険における進路選択のようなものです！

### 順次実行 - まっすぐな道を進む 🚶‍♂️

最も基本的な制御フローは、ステップを順番に実行すること。いわば「まずAの村に行って、次にBの洞窟に行って、最後にCの山に登る」という直線的な冒険ルートです：

```typescript
myWorkflow.step(stepOne).then(stepTwo).then(stepThree).commit();
```

読み方は簡単！「stepOneをやって、それからstepTwoをやって、最後にstepThreeをやる」というだけです。シンプルですね！😊

### 並列実行 - 魔法の多重詠唱 👯‍♂️

独立したタスクを同時に実行する場合は、「分身の術」のように2つのことを同時に進めることができます：

```typescript
myWorkflow.step(fetchUserData).step(fetchOrderData);
```

これは「ユーザーデータを取得しながら、同時に注文データも取得する」という指示。まるで「Aさんは村へ食料を買いに行って、Bさんは同時に森へ薬草を集めに行く」という作戦ですね！時間の節約にもなります。⏱️

### 条件分岐 - 魔法の分かれ道 🔍

魔法の道が二股に分かれた時、どちらへ進むか？条件に基づいて異なる魔法を使い分けるには、`branch`メソッドを使用します：

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

この例は「もし技術系の冒険者が来たら技術マップを渡し、ビジネス系の冒険者が来たらビジネスマップを渡す」という案内所のような仕組み。相手に合わせた対応ができる賢い魔法ですね！🧠

### 条件付きループ - 魔法の繰り返し 🔄

「目標達成するまで繰り返し呪文を唱える」というような、条件に基づいてステップを繰り返し実行する魔法。`until`または`while`メソッドを使うとこんな技も使えます：

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

これは「宝箱が開くまで鍵を回し続ける」ような魔法です。目標達成（値が10以上）になるまで、同じ魔法（incrementStep）を繰り返し唱えます。根気強い魔法使いになれますね！🔑

`while`メソッドは別の書き方。こちらは「条件が真である間、ステップを繰り返す」という魔法です：

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

「体力がある間は走り続ける」というような感じですね。体力（値）が10未満の間は走り（incrementStep）続けます。この違いを理解すると、もっと柔軟な魔法が使えるようになりますよ！🏃‍♂️

## データの流れ - 魔法のエネルギー循環 💫

ワークフロー内での魔力（データ）の受け渡しは、エージェントシステムの重要な側面です。まるで魔法使い同士が魔力を受け渡すような、美しい連携プレイの秘密を紐解きましょう！

### ステップ間でのデータ受け渡し - 魔力のバトン 🏁

前のステップの魔法の結果（出力）を次のステップの材料（入力）として使うことができます：

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

これは「Aさんが集めた材料をBさんが料理する」というチームワーク。`fetchDataStep`で材料（データ）を集めて、その材料を`processDataStep`で料理（処理）しています。`getStepPayload`というメソッドで、前のステップの成果物を受け取ることができるんです。まるでバトンリレーのようですね！🏃‍♀️🏃‍♂️

### 変数マッピング - 魔法の設計図を明確に 🗺️

より明示的なデータの受け渡し、つまり「この材料はこっちに、あの材料はあっちに」と明確に指示するには、変数マッピングが便利です：

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

このやり方だと「Aステップの〇〇という結果を、Bステップの□□という材料として使う」と明示的に指定できます。魔法の設計図がより明確になるので、複雑な魔法でも混乱しにくくなりますよ！😉

## 変数 - 魔法の記憶術 💭

ワークフロー内で変数という「メモ」を使うことで、異なるステップ間で情報を共有したり、計算結果を保存したりできます。まるで冒険中のメモ帳のような存在です！

### ワークフロー変数の使用 - 冒険の記録をつける 📒

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

これは「冒険者の名前をメモしておいて、後で挨拶する」というシナリオ。`setVariableStep`で冒険者の名前に基づいた挨拶文を変数（メモ）として保存し、後の`useVariableStep`でそのメモを取り出して使っています。冒険の途中で「そういえば最初に聞いた名前は...」と思い出せる魔法ですね！📝

## 一時停止と再開 - 魔法の休憩と復帰 ⏸️▶️

長い魔法詠唱の途中で休憩が必要になったり、外部からの入力を待つ必要があったりする場合、一時停止と再開の機能が便利です。まるで「続きはまた明日」と冒険を一時中断するようなものですね。

### ワークフローの一時停止 - 魔法の休憩時間 ⏸️

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

この例では「魔法を途中まで詠唱して、いったん休憩し、ユーザーからの入力を待ってから再開する」という高度な魔法を使っています。`suspendStep`で「ちょっと待って、次に進む前にユーザーの意見が必要！」と一時停止し、入力を待ちます。その後、ワークフローが再開されると、`resumeStep`が「さて、休憩前は何をしていたっけ？新しい情報は？」と確認して処理を続行します。長い冒険の途中で一休みするような感覚ですね！☕

### ワークフローの再開 - 魔法の復帰 ▶️

一時停止した魔法を再開するには、`resume`メソッドという「さあ、続きをやろう！」の呪文を使います：

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

また、ワークフローの状態を監視して自動的に再開処理を行うこともできます。「休憩時間が終わったら自動的に仕事に戻る」ような便利な魔法です：

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

これは「魔法が中断されたら自動的に検知して、必要な材料を補充して再開する」というような監視魔法です。とても便利ですね！🔍

## 動的ワークフロー - 進化する魔法の設計図 🌱

状況に応じてワークフローの構造自体が変化する、生きた魔法の設計図のような動的ワークフローも作成できます。まるで冒険の途中で地図そのものが変化するような、高度な魔法です！

### 動的ステップの生成 - 魔法のレシピを即興で作る 💫

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

これは「冒険の計画を立てる段階で、どんな立ち寄り先が必要かリストアップし、そのリストに基づいて実際の旅程を組み立てる」というような魔法です。入力データに基づいて動的にステップを生成し、それらのステップに対応する処理を実行しています。臨機応変に対応できる柔軟な魔法使いの証ですね！🌟

### ワークフローファクトリ - 魔法工房での即興創作 🏭

さらに高度な例として、実行時に全く異なるワークフローをその場で設計する「魔法工房」のような機能も実装できます：

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

これは魔法使いの中でも最上級の技！「その場の状況を見て、完全に新しい魔法の詠唱法を即興で作り出す」という神業です。入力パラメータを見て「これは簡単な呪文で解決できるな」または「複雑な多段階魔法が必要だな」と判断し、その場で適切な魔法の設計図を生み出し実行します。究極の応用力と言えるでしょう！🧙‍♂️✨

## 実践例：マルチエージェントワークフロー - 魔法使いたちの協演 🎭

最後に、複数の魔法使い（エージェント）を組み合わせた壮大な魔法の実践例を見てみましょう。この例では、文章を書くのが得意な「コピーライター魔法使い」と、文章を磨くのが得意な「エディター魔法使い」を順番に呼び出し、素晴らしいコンテンツを創造します。

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

この例は、まるで出版社での共同作業のよう！まずコピーライター魔法使いがトピックに基づいて記事の下書きを作成し、次にエディター魔法使いがその下書きを磨き上げて輝かせます。それぞれの魔法使いが得意分野で力を発揮し、一人では作れないような素晴らしい作品が出来上がります。チームワークの魔法ですね！👨‍👩‍👧‍👦✨

## まとめ - 魔法の旅を振り返って 🌈

おめでとうございます！この章では、Mastraにおけるワークフローという魔法の設計図の概念と実装方法について冒険してきました：

- ワークフローの基本構造とその目的 📜
- ステップの定義と実装方法 🔍
- 制御フローの実現（順次実行、条件分岐、ループ）🔀
- ステップ間のデータの流れと変数の使用 💫
- ワークフローの一時停止と再開 ⏸️▶️
- 動的ワークフローの実装 🌱
- 複数エージェントを組み合わせた実践例 👥

あなたはこれで、単なる魔法の杖の使い方だけでなく、魔法の設計図を描く能力まで手に入れました！これにより、複雑な問題を構造化して解決し、エージェントたちの力を最大限に引き出すことができるようになります。

次の章では、魔法使いの知識を拡張する「RAG（Retrieval Augmented Generation）」という特別な技術について学んでいきましょう。RAGを使えば、エージェントはさらに膨大な知識ベースを活用して、より賢く、より役立つ助言ができるようになります！

さあ、次なる冒険へ出発しましょう！🚀
