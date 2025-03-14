---
title: "Getting Started（はじめに）"
---

# Mastra開発への第一歩を踏み出そう！

さあ、Mastraの世界へようこそ！これからあなたは「AIエージェントを作る」というエキサイティングな旅に出ます。少し難しく感じるかもしれませんが、心配無用！この章では誰でも安心して始められるよう、環境構築から丁寧に案内します。

一緒にワクワクしながら準備を整えていきましょう。

## インストール

MastraはTypeScriptベースのフレームワークで、Node.jsの環境で動きます。まずは以下の準備をササッと済ませましょう。

### 必要な環境を準備しよう

- **Node.js**がインストールされていることを確認（まだの方は[こちら](https://nodejs.org/)から最新版をゲット！）
- **LLMプロバイダのAPIキー**を準備（GoogleのジェミニAPIキーがおすすめ！）

準備はOK？それじゃあ早速Mastraをセットアップしましょう！

### Mastraプロジェクトの作り方（超カンタン！）

Mastraの開発環境を一瞬で立ち上げるために、公式の便利コマンドを使います。ターミナルで以下のどれかを入力しましょう。

```bash
npm create mastra@latest
# または
yarn create mastra
```

すると、親切なウィザードが色々質問してきます。難しくないので楽しんで進めましょう。

1. **プロジェクト名を決定！** 好きな名前を決めちゃってください。
2. **コンポーネントを選択**: 初心者の方は全部入りがオススメ！
3. **LLMプロバイダの設定**: Googleのジェミニ（特にFlash 2.0）が最もコスパが良くて高性能です。
4. **サンプルコードを含める？**: 最初は「Yes」でサンプルを参考にしながら進めましょう。

これだけで、開発環境が整います！

### 実際の設定の流れ

実際にコマンドを実行すると、下記のようなインタラクティブなやり取りが始まります。ここでは最も一般的な選択肢を選んでいます：

```
npm create mastra@latest

Need to install the following packages:
create-mastra@0.1.9
Ok to proceed? (y) y


> npx
> create-mastra

┌  Mastra Create
│
◇  What do you want to name your project?
│  github-cursor-rules-agent
│
◇  Project created
│
◇  npm dependencies installed
│
◇  mastra installed
│
◇  @mastra/core installed
│
◇  .gitignore added
│
└  Project created successfully


┌  Mastra Init
│
◇  Where should we create the Mastra files? (default: src/)
│  src/
│
◇  Choose components to install:
│  Agents, Workflows
│
◇  Add tools?
│  Yes
│
◇  Select default provider:
│  OpenAI
│
◇  Enter your openai API key?
│  Skip for now
│
◇  Add example
│  Yes
│
◇  
│
◇   ─────────────────────────────────────────────────────────╮
│                                                            │
│                                                            │
│        Mastra initialized successfully!                    │
│                                                            │
│        Add your OPENAI_API_KEY as an environment variable  │
│        in your .env.development file                       │
│                                                            │
│                                                            │
├────────────────────────────────────────────────────────────╯
│
└  
   To start your project:

    cd github-cursor-rules-agent
    npm run dev
```

> **注意**: 上記のセットアップでは「Select default provider:」でOpenAIを選択していますが、**GoogleのGemini API（特にFlash 2.0モデル）**が実はコストパフォーマンスと速度の面で最もおすすめです。ただし、現在のMastraのセットアップウィザードではデフォルトでOpenAIが表示されると思います。この選択を行った後、プロジェクト内のコードを少し修正してGeminiに切り替えることをお勧めします。Geminiを使用するための設定は後ほど説明します。

### APIキーの設定

プロジェクトが準備できたら、最後にAPIキーを設定しましょう。プロジェクト内の`.env`ファイルを開いて、自分のAPIキーを書き込んでください。

```
# Googleジェミニの例
GOOGLE_GENERATIVE_AI_API_KEY=あなたのGoogleジェミニAPIキー
```

絶対にGitHubなどにアップしないよう注意！`.env`はちゃんと`.gitignore`に含まれているはずですが、一応確認してくださいね。

### 開発サーバーを動かしてみよう

準備が整ったら、さっそくMastraの開発サーバーを動かしてみましょう！ターミナルで次のコマンドを打つだけです。

```bash
npm run dev
```

すぐに`http://localhost:4111`が立ち上がり、エージェントがあなたを待っています。

## プロジェクト構造を攻略しよう

Mastraプロジェクトを自由自在に使うためには、構造を把握することが重要です。まずはざっくりと構造を見てみましょう。

```
プロジェクトルート/
├── src/
│   ├── mastra/
│   │   ├── agents/
│   │   ├── workflows/
│   │   ├── tools/
│   │   └── index.ts
│   └── server/
├── public/
├── node_modules/
├── .env
├── package.json
├── tsconfig.json
└── tsconfig.json
```

### 各ディレクトリの役割

- **agents/**: エージェントを定義します。AIにどんな仕事を任せたいか、ここで決まります。
- **tools/**: エージェントが利用するツールをカスタムで作ります。API呼び出しやデータ処理など何でも可能！
- **workflows/**: より複雑な処理の流れを組み立てます。
- **index.ts**: ここでエージェントやワークフローをMastraに登録します。

### サンプルコードをどんどん活用しよう！

プロジェクトを作る時に「サンプルコードを含める」を選ぶと、基本的なコードが自動で付いてきます。これを参考に、あなたのオリジナルエージェントを作り始めてみましょう！

## 次はいよいよ実践です！

これでMastra開発への扉が開きました！次の章では、実際にGitHubリポジトリを解析する面白くて実用的なエージェントを一緒に作ります。

準備はOKですか？さぁ、エキサイティングなMastraの世界に飛び込みましょう！