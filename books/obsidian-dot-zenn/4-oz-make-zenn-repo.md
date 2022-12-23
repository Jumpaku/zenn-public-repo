---
title: "保管庫と Zenn リポジトリの作成"
cssclass: zenn
date: 2022-12-22
modified: 2022-12-22
url: "https://zenn.dev/estra/articles/oz-make-zenn-repo"
AutoNoteMover: disable
tags: [" #type/zenn/book #obsidian  "]
aliases: OZ本『保管庫と Zenn リポジトリの作成』
---

# このチャプターについて

このチャプターでは、Obsidian の保管庫に Zenn のリポジトリを作成して運用していく方法について解説していきます。

# Zenn のリポジトリを作成しよう

Zenn リポジトリの基本的な作成方法と連携方法については以下の Zenn 公式記事で紹介されています。

https://zenn.dev/zenn/articles/connect-to-github

ここでは、Obsidian において Zenn リポジトリを作成する方法にフォーカスして解説していきます。

## リポジトリ配置用フォルダの作成

まずはリポジトリを配置するためのディレクトリを保管庫内に作成します。

Zenn に限らず、ブログなどのリポジトリを Obsidian 内部に構築する場合には特定のディレクトリに集約して配置しておくと良いです。

これは同期プラグインでのファイル同期と git 管理しているリポジトリの管理を分離しておくためです。

筆者の場合には以下のように SubVault というフォルダを作成して、Zenn や他のブログ用のリポジトリを一括で配置して管理しています。

```sh
❯ exa --tree --level=1 ./SubVault/
./SubVault
├── deno-blog
├── GistDir
├── hugo-repo
├── zenn-private-book
└── zenn-repo
```

## 同期プラグインを利用している場合にはフォルダを除外しよう

同期プラグインを使っている場合には、上記のリポジトリ配置用のディレクトリを同期から除外するように設定しておきましょう。「コアプラグイン」→「同期」→「選択的同期」→「除外フォルダ」→「管理」から除外したいフォルダ名を設定します。

![img](/images/oz/img_oz-selective-sync.jpg)

これでリポジトリ内のファイルはプラグインによる同期がされないようになりました。

## npm ではなく pnpm や deno などを使う

リポジトリを用意するための準備ができたので作成していきますが、まず注意点として、なんらかのライブラリやパッケージをリポジトリ内で利用する際には npm ではなく [pnpm](https://pnpm.io) や [deno](https://deno.land) などの `node_modules` フォルダをなるべく作る必要のないツールを使うのをおすすめします。

というのも npm を使うと深いネストのある `node_moduels` フォルダがディレクトリに作成されてしまい、ノートの検索時に利用しない大量のフォルダが無駄にサジェストされたり、「ファイルを別のフォルダに移動」のコマンド実行時に探しようのないネストの深い場所に誤ってノートが飛ばされてします可能性があるからです。

執筆環境で利用したい Textlint などのパッケージなら pnpm でも deno でも利用できるので、そちらのツールを使うようにしましょう。

## 新しくローカルから作成する場合

zenn-cli を使う場合には以下のコマンドでローカルインストールします。

```sh
❯ pnpm install zenn-cli
```

[pnpm exec](https://pnpm.io/cli/exec) コマンドで `npx` の一部に相当するシェルコマンド実行ができます。

```sh
❯ pnpm exec zenn init

  🎉  Done!
  早速コンテンツを作成しましょう

  👇  新しい記事を作成する
  $ zenn new:article

  👇  新しい本を作成する
  $ zenn new:book

  👇  投稿をプレビューする
  $ zenn preview

```

この場合には以下のようなディレクトリ構造が展開されます。

```sh
❯ exa --tree --level=3 -a
.
├── .gitignore
├── articles
│  └── .keep
├── books
│  └── .keep
├── node_modules
│  ├── .bin
│  │  └── zenn
│  ├── .modules.yaml
│  ├── .pnpm
│  │  ├── lock.yaml
│  │  └── zenn-cli@0.1.134
│  └── zenn-cli -> .pnpm/zenn-cli@0.1.134/node_modules/zenn-cli
├── package.json
├── pnpm-lock.yaml
└── README.md
```

`node_modules` フォルダさえ作らずに、使い捨てのプログラムとして実行したい場合にはこれまた `npx` 相当である [pnpm dlx](https://pnpm.io/cli/dlx) コマンドを実行することで可能です。

```sh
❯ pnpm dlx zenn-cli init
.../Library/pnpm/store/v3/tmp/dlx-79668  |   +1 +
.../Library/pnpm/store/v3/tmp/dlx-79668  | Progress: resolved 1, reused 1, downloaded 0, added 1, done

  🎉  Done!
  早速コンテンツを作成しましょう

  👇  新しい記事を作成する
  $ zenn new:article

  👇  新しい本を作成する
  $ zenn new:book

  👇  投稿をプレビューする
  $ zenn preview

```

この場合には以下のような簡素なディレクトリ構造が展開されます。

```sh
❯ exa --tree --level=3 -a
.
├── .gitignore
├── articles
│  └── .keep
├── books
│  └── .keep
└── README.md
```

:::message alert
この場合には VSCode の Zenn プラグインなどが利用できないので注意してください。
:::

## Textlint を入れよう

あとで紹介する Obsidian Linter プラグインでもフォーマットはできますが、より詳細なリンター/フォーマットルールの設定や校正を行いたい場合には Textlint を開発依存として入れます。

```sh
❯ pnpm install -D textlint
```

:::message
pnpm を利用するとグローバルにある store と呼ばれる場所にパッケージが保存されて、プロジェクト間で使い回されるので、データ量の節約ができます。
:::

Textlint で使用する以下のいくつかのプリセットルールも入れておきましょう。

1. [textlint-ja/textlint-rule-preset-JTF-style: JTF日本語標準スタイルガイド for textlint.](https://github.com/textlint-ja/textlint-rule-preset-JTF-style)
2. [textlint-ja/textlint-rule-preset-ja-spacing: スペース周りのスタイルを扱うtextlintルール集](https://github.com/textlint-ja/textlint-rule-preset-ja-spacing)
3. [textlint-ja/textlint-rule-preset-ja-technical-writing: 技術文書向けのtextlintルールプリセット](https://github.com/textlint-ja/textlint-rule-preset-ja-technical-writing)

```sh
❯ pnpm install -D \
  textlint-rule-preset-ja-spacing \
  textlint-rule-preset-ja-technical-writing \
  textlint-rule-spellcheck-tech-word
```

Textlint の設定用ファイル `.textlintrc` なども作成します。

```sh
❯ pnpm exec textlint --init
```

上記コマンドで設定ファイル `.textlintrc` が作成されるので、プレセットに関してのルールなどを設定すれば、textlint が利用できるようになります。

```json:サンプル
{
  "filters": {},
  // 以下設定ルール(trueにしているもののデフォルト設定が有効になっているので、それぞれ追加で部分的に設定していく必要がある)
  "rules": {
    // スペース
    "preset-ja-spacing": false, // スペースは今回見ない
    // JTFのスタイルガイド 
    "preset-jtf-style": {
      "1.1.3.箇条書き": false, // 箇条書きの文末の読点をチェックしない
      "2.2.2.算用数字と漢数字の使い分け": false, // 数えられるやつでも漢数字に変更
      "3.2.カタカナ語間のスペースの有無": false, // 変更
      "4.2.7.コロン(：)": false, // コロンは半角に変更
      "4.3.1.丸かっこ（）": false, // 丸括弧は半角に変更
      "4.3.2.大かっこ［］": false, // 大括弧は半角に変更
      "4.3.3.かぎかっこ「」": false, // かぎかっこは半角に変更
      "4.3.4.二重かぎかっこ『』": false, // 半角に変更
    },
    // テクニカルライティング
    "preset-ja-technical-writing": {
      "sentence-length": false, // 行長さをチェックしないように変更
      "no-exclamation-question-mark": false, // 感嘆符と疑問符を使用に変更
      "no-hankaku-kana": false, // 半角カナの使用をチェックしない
    },
    // スペルチェッカーは無効化 jtfと衝突する
    "spellcheck-tech-word": false,
    // 辞書(表記ゆれを防ぐ) 適宜設定する
    "prh": {
      "rulePaths": ["./prh.yml"]
    }
  }
}
```

以下のプラグインで VSCode 上で警告などが閲覧できるようになります。

https://github.com/taichi/vscode-textlint

さて、執筆環境を作る準備ができましたね。
