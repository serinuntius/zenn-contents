---
title: "Workflows（ワークフロー）"
---

# ワークフローの力を極めよう！ 🔄✨

前章ではMastraエージェントの基本構成要素や機能という、重要な基礎知識について学びましたね！素晴らしい進歩です。これらの知識を武器に、今回はさらに一歩進んで、エージェントに具体的な行動手順を教える「ワークフロー」について探検していきましょう。エージェントの行動パターンを自在に設計できるようになりますよ！

## ワークフローの概要 - 処理の設計図 📜

ワークフローとは、エージェントが実行すべき処理の流れを構造化して定義した、いわば「行動の地図」のようなものです。旅で言えば、「まず駅に行って、次にホテルを探して、最後に観光地に向かう」といった具体的な行程表のようなもの。複雑なタスクを小さなステップに分解し、それらをどのような順序や条件で実行するかを明確に指示できるんです！

### なぜワークフローが必要なのか 🤔

エージェントは単純なタスクであれば単体で十分対応できますが、以下のような場合には、ワークフローという設計図が非常に役立ちます：

1. **複雑な処理の分解** 🧩: 大きなタスクを小さなステップに分割することで、処理が管理しやすくなります
2. **処理の分岐点の実現** 🌳: 「もし〇〇なら□□の処理、そうでなければ△△の処理」という状況判断ができます
3. **データの受け渡し** 💫: ステップ間でデータをスムーズに受け渡せます
4. **エラー対策** 🛡️: 処理中の問題を適切に捕捉し対応できます
5. **並列処理** ⚡: 独立したタスクを同時に実行することで効率アップ！

旅行で例えるなら、「雨が降ったら傘を持って行く」「混雑している場合は別のルートを選ぶ」などの状況に応じた計画が立てられるということです。賢い旅行者になるための必須スキルですね！

### ワークフローの基本構造 - 処理の骨組み 🧠

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

このコードは少し複雑に見えるかもしれませんが、心配いりません！基本的には「ワークフロー」という旅の大まかな計画を立て、その中に「ステップ」という個別の立ち寄りポイントを設定しているだけなんです。`triggerSchema`はこの旅に必要な情報リスト、`Step`オブジェクトは旅の各立ち寄りポイントで何をするかを記した指示書のようなものですね！✨

## ステップの定義 - 処理レシピを書く 📝

ステップはワークフローの最小実行単位、言わば処理レシピの1ステップです。「卵を割る」「小麦粉を入れる」のような、レシピの各手順をステップとして定義していきます。

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

### エージェントを呼び出すステップ - 専門家に協力を求める 🧚‍♀️

ワークフロー内から別のエージェントを呼び出すステップを定義することもできます。まるで旅行中に専門家の助けを借りるようなものですね：

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

このステップでは、「私には文章を書くのが苦手だから、文章の得意な友達に手伝ってもらおう！」というようなことをやっています。旅行の途中で専門のガイドに相談するようなものですね。ワークフローの中で専門家（エージェント）の助けを借りることで、より素晴らしい結果が得られます！🌟

## 制御フロー - 処理の道筋を設計する 🗺️

ワークフローの処理の流れをコントロールするための仕組み、つまり処理の道筋をどう設計するかについて見ていきましょう。旅行における進路選択のようなものです！

### 順次実行 - まっすぐな道を進む 🚶‍♂️

最も基本的な制御フローは、ステップを順番に実行すること。いわば「まずAの場所に行って、次にBの場所に行って、最後にCの場所に行く」という直線的な旅行ルートです：

```typescript
myWorkflow.step(stepOne).then(stepTwo).then(stepThree).commit();
```

読み方は簡単！「stepOneをやって、それからstepTwoをやって、最後にstepThreeをやる」というだけです。シンプルですね！😊

### 並列実行 - 処理の同時進行 👯‍♂️

独立したタスクを同時に実行する場合は、「分担作業」のように2つのことを同時に進めることができます：

```typescript
myWorkflow.step(fetchUserData).step(fetchOrderData);
```

これは「ユーザーデータを取得しながら、同時に注文データも取得する」という指示。まるで「Aさんは食料を買いに行って、Bさんは同時に宿を予約する」という作戦ですね！時間の節約にもなります。⏱️

### 条件分岐 - 処理の分かれ道 🔍

処理の道が二股に分かれた時、どちらへ進むか？条件に基づいて異なる処理を選択するには、`branch`メソッドを使用します：

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

この例は「もし技術系の訪問者が来たら技術的な資料を渡し、ビジネス系の訪問者が来たらビジネス向け資料を渡す」という案内所のような仕組み。相手に合わせた対応ができる賢い仕組みですね！🧠

### 条件付きループ - 処理の繰り返し 🔄

「目標達成するまで繰り返し処理を行う」というような、条件に基づいてステップを繰り返し実行する技術。`until`または`while`メソッドを使うとこんなこともできます：

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

これは「目的地に着くまでナビの指示に従って進み続ける」ような処理です。目標達成（値が10以上）になるまで、同じ処理（incrementStep）を繰り返します。根気強い作業ですね！🔑

`while`メソッドは別の書き方。こちらは「条件が真である間、ステップを繰り返す」という処理です：

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

「体力がある間は走り続ける」というような感じですね。体力（値）が10未満の間は走り（incrementStep）続けます。この違いを理解すると、もっと柔軟な処理が設計できるようになりますよ！🏃‍♂️

## データの流れ - 情報の循環 💫

ワークフロー内でのデータの受け渡しは、エージェントシステムの重要な側面です。データがステップ間でどのように流れるかを見ていきましょう！

### ステップ間のデータ共有 - 情報のバケツリレー 🪣

ワークフロー内の各ステップは、前のステップの出力を入力として利用できます。まるでバケツリレーのように、データを次々と受け渡していくのです：

```typescript
import { Step, Workflow } from "@mastra/core";

// ステップ1: 名前を加工
const formatNameStep = new Step({
  id: "formatName",
  execute: async ({ context }) => {
    const name = context.machineContext?.triggerData?.name || "名無し";
    return { formattedName: name.toUpperCase() };
  },
});

// ステップ2: 挨拶を生成
const createGreetingStep = new Step({
  id: "createGreeting",
  execute: async ({ context }) => {
    // 前のステップの結果を取得
    const formattedName = context.getStepResult(formatNameStep)?.formattedName;
    return { greeting: `こんにちは、${formattedName}さん！` };
  },
});

// ワークフローの定義
const greetingWorkflow = new Workflow({ name: "greeting-workflow" });
greetingWorkflow.step(formatNameStep).then(createGreetingStep).commit();
```

「うわっ、これ少し複雑...」と感じるかもしれませんが、実は単純な流れです。「名前を大文字に変換して」→「それを使って挨拶文を作る」というだけのことです。前のステップの結果を`context.getStepResult()`で取得できるのがポイントです！

### ワークフローの入出力 - 処理の始まりと終わり 🔄

ワークフローは外部から入力を受け取り、最終的な結果を出力します。この入出力の流れは、関数やAPIと同じような仕組みです：

```typescript
import { Workflow } from "@mastra/core";
import { z } from "zod";

// ワークフローの入力スキーマを定義
const simpleWorkflow = new Workflow({
  name: "simple-workflow",
  triggerSchema: z.object({
    name: z.string().describe("ユーザー名"),
    age: z.number().describe("年齢"),
  }),
});

// 処理ステップを追加...

// ワークフローを実行
const result = await simpleWorkflow.trigger({
  name: "山田太郎",
  age: 30,
});

console.log("ワークフロー結果:", result);
```

ワークフローは明確な「入口」と「出口」を持っていて、入力データの形式を`triggerSchema`で定義し、最終的な結果はワークフローの実行結果として返されます。API設計とよく似ていますね！

### コンテキスト - 処理の記憶 🧠

ワークフローが実行されている間、すべてのデータは「コンテキスト」というメモリに保存されます。このコンテキストには以下のようなデータが含まれます：

1. **triggerData** - ワークフロー起動時の入力データ
2. **各ステップの結果** - 既に実行されたステップの出力
3. **システム情報** - タイムスタンプ、実行IDなど

以下のように、様々な方法でコンテキストにアクセスできます：

```typescript
// コンテキストからデータを取得する例
execute: async ({ context }) => {
  // 1. ワークフロー起動時の入力データにアクセス
  const name = context.machineContext?.triggerData?.name;
  
  // 2. 前のステップの結果にアクセス
  const previousResult = context.getStepResult(previousStep);
  
  // 3. 特定のステップの特定フィールドにアクセス
  const age = context.getStepOutput(userInfoStep, 'age');
  
  return { message: `${name}さん、${age}歳の方ですね！` };
}
```

コンテキストは「ワークフローの記憶」のようなもの。すべてのステップが共有するノートのようなイメージです。この仕組みを活用すれば、スムーズなデータの流れを実現できますよ！

## 実用的なワークフロー例 - 実践演習 💼

学んだ知識を活かして、実用的なワークフローの例を見てみましょう。以下は「ユーザーからの問い合わせを処理する」ワークフローの例です：

```typescript
import { Workflow, Step } from "@mastra/core";
import { z } from "zod";
import { openai } from "@ai-sdk/openai";

// 問い合わせ内容を分類するステップ
const classifyInquiryStep = new Step({
  id: "classifyInquiry",
  execute: async ({ context }) => {
    const inquiry = context.machineContext?.triggerData?.message;
    const agent = new Agent({
      model: openai("gpt-4o"),
      instructions: "問い合わせを「技術的質問」「料金関連」「アカウント問題」「その他」のいずれかに分類してください"
    });
    
    const result = await agent.generate(inquiry);
    let category = "その他";
    
    if (result.text.includes("技術的")) category = "技術的質問";
    else if (result.text.includes("料金")) category = "料金関連";
    else if (result.text.includes("アカウント")) category = "アカウント問題";
    
    return { category };
  }
});

// 各カテゴリに対応するステップを用意
const techSupportStep = new Step({
  id: "techSupport",
  execute: async ({ context }) => {
    const inquiry = context.machineContext?.triggerData?.message;
    // 技術的な回答を生成...
    return { response: "技術的なご質問ありがとうございます。..." };
  }
});

const billingStep = new Step({
  id: "billing",
  execute: async ({ context }) => {
    // 料金に関する回答を生成...
    return { response: "料金に関するご質問ありがとうございます。..." };
  }
});

const accountStep = new Step({
  id: "account",
  execute: async ({ context }) => {
    // アカウントに関する回答を生成...
    return { response: "アカウントに関するご質問ありがとうございます。..." };
  }
});

const otherInquiryStep = new Step({
  id: "otherInquiry",
  execute: async ({ context }) => {
    // その他の質問への回答を生成...
    return { response: "お問い合わせありがとうございます。..." };
  }
});

// 最終的な返信を整形するステップ
const formatResponseStep = new Step({
  id: "formatResponse",
  execute: async ({ context }) => {
    // 前のステップの応答を取得
    const response = context.getStepOutput(techSupportStep, 'response') ||
                     context.getStepOutput(billingStep, 'response') ||
                     context.getStepOutput(accountStep, 'response') ||
                     context.getStepOutput(otherInquiryStep, 'response');
    
    return {
      formattedResponse: `
        お問い合わせありがとうございます。
        
        ${response}
        
        他にご質問があればお気軽にお問い合わせください。
        
        サポートチーム
      `
    };
  }
});

// ワークフローを組み立てる
const customerSupportWorkflow = new Workflow({
  name: "customer-support",
  triggerSchema: z.object({
    message: z.string(),
    email: z.string().email()
  })
});

customerSupportWorkflow
  .step(classifyInquiryStep)
  .branch(({ context }) => {
    return context.getStepResult(classifyInquiryStep)?.category;
  })
  .when("技術的質問", workflow => workflow.step(techSupportStep))
  .when("料金関連", workflow => workflow.step(billingStep))
  .when("アカウント問題", workflow => workflow.step(accountStep))
  .when("その他", workflow => workflow.step(otherInquiryStep))
  .then(formatResponseStep)
  .commit();
```

このワークフローは、以下のような流れで動作します：

1. ユーザーからの問い合わせメッセージを受け取る
2. AIを使って問い合わせの内容を分類する
3. 分類に基づいて適切な専門担当者（ステップ）に転送
4. 回答を生成
5. 共通のフォーマットで返信を整形

これは、実際のカスタマーサポートシステムの縮小版のように機能します。人間のオペレーターが行うような「問い合わせの内容を理解して、適切な部門に振り分ける」という判断をAIが行い、それぞれの専門部署が回答するという流れを自動化しているのです。

## エラーハンドリング - 問題への対処法 🛡️

ワークフローの実行中に問題が発生しても、適切に対応できるようにエラーハンドリングの仕組みを組み込むことができます：

```typescript
import { Workflow, Step } from "@mastra/core";

// エラーが発生するかもしれないステップ
const riskyStep = new Step({
  id: "riskyStep",
  execute: async () => {
    const random = Math.random();
    if (random < 0.5) {
      throw new Error("何かがうまくいきませんでした");
    }
    return { success: true };
  }
});

// エラーハンドリングステップ
const errorHandlerStep = new Step({
  id: "errorHandler",
  execute: async ({ context }) => {
    const error = context.machineContext?.error;
    console.log("エラーを処理しています:", error);
    // ここでエラーログを記録したり、代替処理を実行したりできます
    return { 
      errorHandled: true,
      message: "エラーを処理しました"
    };
  }
});

// ワークフローの定義
const workflowWithErrorHandling = new Workflow({ name: "error-handling" });

workflowWithErrorHandling
  .step(riskyStep)
  .catch(errorHandlerStep)
  .commit();
```

この`catch`メソッドにより、問題が発生したときに特定のステップ（この場合は`errorHandlerStep`）が実行されます。これによって、エラーの記録、ユーザーへの通知、代替処理の実行など、様々な対応が可能になります。まるで道路の迂回路のように、メインルートで問題が発生したときの代替ルートを用意しておくのです！

## まとめ - ワークフローの力 🌟

Mastraのワークフローは、AIエージェントにとって強力な機能です。複雑なタスクを小さなステップに分割し、条件分岐や並列処理のような高度な制御フローを導入することで、エージェントはより賢く、効率的に、そして柔軟に動作するようになります。

まるで複雑な料理レシピや詳細な旅行計画のように、適切にステップを組み合わせることで、思いもよらない素晴らしい結果を生み出すことができるのです。

ワークフローを使いこなせるようになると、あなたは単なるエージェント開発者から、複雑なAIシステムのアーキテクトへと成長するでしょう！🏗️

次の章では、エージェントの知識を増強する「RAG（検索拡張生成）」について学んでいきます。さらに強力なエージェントを作るための次のステップに進みましょう！
