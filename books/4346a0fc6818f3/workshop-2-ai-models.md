---
title: "第4章：AIモデルの設定と活用（models）"
---

# AIモデルの設定と活用方法を学ぼう！🧠✨

前章では、Mastraフレームワークの中心となるindex.tsファイルについて学びました。おめでとうございます！最初の一歩を踏み出しましたね。さあ、今度はもっとワクワクする部分へと進みましょう—AIモデルの設定と活用です！

AIモデルは、エージェントの「頭脳」とも言える重要な部分。どのモデルを選び、どう設定するかによって、エージェントの能力や特性が大きく変わります。この章では、豊富なオプションの中から最適なモデルを選ぶ方法と、それらを最大限に活用するテクニックを学んでいきましょう！

## AIモデルって何がある？どう選べばいい？🤔

現在、数多くのAIモデルが利用可能ですが、主に以下の3つの大手プロバイダーが人気です：

1. **Google Gemini** - コストパフォーマンスに優れ、応答速度も速い
2. **Anthropic Claude 3.7 Thinking** - プログラミングに圧倒的に強い、Toolの使い方も上手い
3. **OpenAI GPT** - バランスの取れた性能と幅広いAPIオプション

大切なのは、あなたのプロジェクトに最適なモデルを選ぶこと。予算、速度、精度、文脈理解能力などを考慮して選びましょう。

## モデルの設定方法 - 実際のコードで学ぼう🔧

さっそく、複数のAIモデルを設定する実際のコードを見てみましょう！

```typescript
import { createGoogleGenerativeAI } from "@ai-sdk/google";
import { createOpenAI } from "@ai-sdk/openai";

// Google Gemini AIプロバイダーの作成
export const google = createGoogleGenerativeAI({
    apiKey: process.env.GOOGLE_API_KEY || "",
});

// エンべディングモデルのインスタンス
export const googleEmbeddingModel =
    google.textEmbeddingModel("text-embedding-004");

// anthropic/claude-3.7-sonnet:thinking
// Claude 3.7 Thinking モデルのインスタンス
export const claudeThinkingModel = "anthropic/claude-3.7-sonnet:thinking";

const openai = createOpenAI({
    apiKey: process.env.OPENROUTER_API_KEY || "",
    baseURL: "https://openrouter.ai/api/v1",
});

// OpenRouter経由でのClaude 3.7 Thinkingモデル指定
const openRouterClaudeThinkingModel = "anthropic/claude-3-7-sonnet:thinking";

// OpenRouter経由でのモデル利用設定
export const openRouter = openai(openRouterClaudeThinkingModel);
```

このコードはかなり面白いことをしています！複数のAIプロバイダーを一つのファイルで設定し、使い分けられるようにしているんです。一つずつ解説していきましょう。

### 1. Google Geminiの設定🌟

```typescript
// Google Gemini AIプロバイダーの作成
export const google = createGoogleGenerativeAI({
    apiKey: process.env.GOOGLE_API_KEY || "",
});
```

これはGoogle Geminiモデルを使用するための設定です。Google CloudのAPIキーを環境変数から読み込んで初期化しています。Geminiモデルは、コストパフォーマンスと速度のバランスが優れているため、多くのユースケースで第一選択肢となります！

### 2. エンベディングモデルの設定🔍

```typescript
// エンべディングモデルのインスタンス
export const googleEmbeddingModel =
    google.textEmbeddingModel("text-embedding-004");
```

エンベディングモデルは、テキストを数値ベクトルに変換するモデルです。これはRAG（Retrieval-Augmented Generation）システムや文書検索など、テキストの意味的類似性を計算する必要がある場合に非常に重要です。Googleの`text-embedding-004`モデルは高性能で経済的な選択肢です。

### 3. Claude 3.7 Thinkingモデルの参照🧠

```typescript
// anthropic/claude-3.7-sonnet:thinking
// Claude 3.7 Thinking モデルのインスタンス
export const claudeThinkingModel = "anthropic/claude-3.7-sonnet:thinking";
```

ここでは、Anthropicの最新モデルである`Claude 3.7 Thinking`を参照しています。このモデルは特に複雑な推論や詳細な分析が必要な場合に強みを発揮します。後ほどAnthropicのSDKと組み合わせて使用することができます。

### 4. OpenRouterを通じたモデルアクセス🌉

```typescript
const openai = createOpenAI({
    apiKey: process.env.OPENROUTER_API_KEY || "",
    baseURL: "https://openrouter.ai/api/v1",
});

// OpenRouter経由でのClaude 3.7 Thinkingモデル指定
const openRouterClaudeThinkingModel = "anthropic/claude-3-7-sonnet:thinking";

// OpenRouter経由でのモデル利用設定
export const openRouter = openai(openRouterClaudeThinkingModel);
```

これが非常に興味深いパートです！OpenRouterというサービスを利用して、OpenAIのインターフェースを通じてClaudeモデルにアクセスしています。これにより、複数のAIプロバイダーのモデルを統一的なインターフェースで利用できるようになります。まさに「一石二鳥」のアプローチですね！

## APIキーの管理 - セキュアな方法で🔐

コード内では、APIキーを環境変数から読み込んでいることに注目してください：

```typescript
apiKey: process.env.GOOGLE_API_KEY || "",
```

```typescript
apiKey: process.env.OPENROUTER_API_KEY || "",
```

この方法には重要な利点があります：

1. **セキュリティ強化** - APIキーをコードに直接書かない
2. **環境ごとの切り替え容易** - 開発環境と本番環境で異なるキーを使用可能
3. **Git管理の安全性** - `.env`ファイルを`.gitignore`に追加することでキーの流出を防止

必ず`.env`ファイルをプロジェクトのルートに作成し、以下のように記述しましょう：

```
GOOGLE_API_KEY=あなたのGoogleAPIキーをここに
OPENROUTER_API_KEY=あなたのOpenRouterAPIキーをここに
```

そして、この`.env`ファイルは絶対にGitHubなどにアップロードしないでください！

## モデル選択のベストプラクティス👌

どのモデルを選ぶべきか迷ったら、以下のポイントを考慮するといいでしょう：

1. **タスクの複雑さ** - 単純な応答ならGemini Flash、複雑な推論や長文処理ならClaudeが適している
2. **応答速度の重要性** - ユーザー体験を重視するならGemini Flashが速い
3. **コスト** - 予算に制約があるならGeminiモデルが一般的にコスト効率が良い
4. **コンテキスト長** - Geminiは異様に長いコンテキスト長を持ち、多数のファイルを一度に処理する必要がある場合に非常に有効
6. **プログラミングタスク** - Claudeは圧倒的にプログラミング理解と生成が得意ですが、その分コストが高い

特にコードベースの解析や生成が重要なプロジェクトでは、Claude 3.7 Thinkingのような高性能モデルを選ぶメリットが大きいでしょう。コードの構造理解、バグの発見、効率的なアルゴリズムの提案など、プログラミング関連のタスクではClaudeの能力が光ります。一方、大規模なコードベースを一度に解析する必要がある場合は、Geminiの広大なコンテキスト長が非常に強力な武器となります。複数のソースファイル、設定ファイル、ドキュメントを同時に参照しながら解析するようなケースでは、Geminiの長いコンテキスト長が作業効率を大幅に向上させるでしょう。

モデル選択は常にタスク、予算、必要な精度のバランスを考慮して行うことが重要です。今回のGitHub解析エージェントでは、コードを深く理解する必要があるためClaude 3.7 Thinkingが適していますが、多くのリポジトリファイルを一度に分析する必要がある場合はGemini Flash 2.0も非常に有力な選択肢となります。

## 実践：モデルを切り替えてみよう🔄

プロジェクトの途中でモデルを切り替えたくなった場合、index.tsから参照するファイルを変更するだけで簡単に切り替えられます。これがモデル設定を別ファイルにした利点の一つです！

例えば、エージェント定義で以下のように参照できます：

```typescript
import { google, openRouter } from "../models";

// Google Geminiを使用
const geminiAgent = new Agent({
  // ...
  model: google("gemini-2.0-flash-001"),
});

// あるいはOpenRouter経由でClaudeを使用
const claudeAgent = new Agent({
  // ...
  model: openRouter,
});
```

このように、同じプロジェクト内で複数のモデルを使い分けることも可能です！

## ここまでのまとめ📝

この章では、AIモデルの設定と活用について学びました：

- Google Gemini、Claude、OpenAIなど様々なモデルの特徴と設定方法
- エンベディングモデルの活用
- OpenRouterを使った複数プロバイダーへのアクセス
- APIキーの安全な管理方法
- タスクに応じたモデル選択のコツ

これでエージェントの「頭脳」部分の準備ができました！次の章では、GitHubリポジトリの操作方法について学び、エージェントが実際に外部サービスと連携する方法を探っていきます。

次のステップへ進む準備はできましたか？GitHubの世界への冒険がもうすぐ始まります！🚀 