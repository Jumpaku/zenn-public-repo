---
title: "fishコマンドの機能的な分類"
emoji: "🏄🏻‍♂️"
type: "tech"
topics: [fish, shell, 初心者]
published: true
date: 2022-03-15
url: "https://zenn.dev/estra/articles/zenn-fish-typeof-commands"
aliases: [記事_fishコマンドの機能的な分類]
tags: " #type/zenn #shell/fish/command  "
---

# はじめに
2022/03/13 にリリースされた `fish v3.4.0` と合わせて公式ドキュメントのコマンドリストのページが更新されました。fish が提供するコマンドのカテゴリ分けが非常に分かりやすくなったため、その内容について説明したいと思います。

:::message
この記事の想定読者は次の方です。
- fish shell 初心者の人
- コマンドをなんとなく使っている人
:::

# 分類の仕方
まず、「コマンド」と一概に言っても次のようにコマンドラインから呼び出される可能性があるものには次の種類があります。

- Function (関数)
- Builtin (ビルトイン)
- External command (外部コマンド)

以前、これらの違いや実体についての解説記事を書きました。

https://zenn.dev/estra/articles/zenn-what-is-command

この分類は、**コマンド概念の全体像を掴む**ためには役立ちますが、初心者にとって実用上はそこまで重要ではありません。

# 機能分類
そういう訳で、今回はコマンドを**機能的な面**から分類して解説していきたいと思います。
と言っても、公式ドキュメントをほぼそのまま参考にしていますので、オリジナルを見たい方は公式ドキュメントの次のページを確認してください。

https://fishshell.com/docs/current/commands.html

※ 解説するカテゴリの中に含まれていないコマンドもあるので注意してください。

## キーワード
以下のものは fish のシンタックスを構築するコアの言語キーワードです。もちろんすべてコマンドですが、これらは「ビルトイン」に属するものです。

- 条件分岐: `if`, `else`
- ループ構築: `for`, `while` 
- ループ制御: `break`, `continue`
- 関数定義: `function`
- 関数からステータスを返却: `return`
- コードブロックの開始と終了: `begin`, `end`
- コマンドの論理的結合: `and`, `or`, `not`
- 変数の値に基づいた複数のブロック作成: `switch`, `case`

これらのコマンドは関数定義で使用することが多いです。

```shell
function mytest -d "This is a test function"
    if not test $argv -ge 1
        echo "Pass arguments"
        return 1
    else
        echo "You typed..." $argv
    end
    
    while true
        set -l loop_exit_flag
        read -l -P "Loop starts, do you wanna exit? [Y/n]" key
        switch "$key"
            case Y y yes
                set loop_exit_flag "true"
                break
            case N n no
                break
            case
                break
        end
        test "$loop_exit_flag" = "true"; and break
    end
end
```

## デコレーション
デコレーション用のコマンドは、他のコマンドの前に付けることでそのコマンドの振る舞いを制御できます。以下のものはすべて「ビルトイン」に属します。

- どの種類のものを実行するか伝える: `command`, `builtin`
- 実行時間を決める: `time`
- 現在のシェルを指定したコマンドで置き換える `exec`

例えば、`cd` をデコレーションすると次のようになります。

```shell:cdのデコレーション
# function の cd を使用
$ cd ../
# builtin の cd を使用
$ builtin cd ../
# exteranl command の cd を使用
$ command cd ../
```

`exec $SHELL -l` でログインシェルを新規化できますが、これも実際にはパラメータ `$SHELL` が展開されて `fish` というコマンド(`/opt/homebrew/bin/fish`)が展開されてプロセスを置き換えています。

```shell
$ exec $SHELL -l
# 上と同等
$ exec fish -l
```

## タスク実行のためのツール
以下は、タスク実行のためのビルトインコマンドです。基本的なタスクはこれらのコマンドを使用して遂行できます。多くが「ビルトイン」に属しますが、中には「関数」であるものも含まれます。

`cd` や `echo` など、コマンドラインで日常的に使用するコマンドの多く含まれます。なので、最初はこれらのコマンドから覚えていくのが良いと思われます。

- カレントディレクトリの変更: `cd`
- アウトプットの生成: `echo`, `printf`
- 変数の定義・検索・削除: `set`
- 入力の読み込み: `read`
- 文字列操作: `string`
- 算術計算: `math`
- 引数の処理: `argparse`
- 引数の数え上げ: `count`
- fish が呼び出すものの種類を特定: `type`
- 条件の検査: `test`
- リスト内に指定したエントリーが含まれるか確認: `contains`
- 略語の管理: `abbr`
- fish コードを文字列やファイルから実行: `eval`, `source`
- アウトプットの色を変更: `set_color`
- shell の情報を取得: `status`
- バインディングの変更: `bind`
- コマンドコマンドの内容の取得と変更: `commandline`
- fish の設定を変更: `fish_config`
- ランダムな数値を生成またはリストから選択: `random`

```shell
# ディレクトリを一個上に登る
$ cd ../
# 文字列を出力
$ echo "test"
test
# 変数の定義(グローバルスコープ)
$ set -g MYVALUE value
# 文字列の結合
$ string join "" $MYVALUE "_plus"
value_plus
# config.fish ファイルの読み込み
$ source ~/.config/fish/config.fish
```

## よく知られた関数
以下のものはカスタマイズするためのコマンドで、使用することで fish の振る舞いを変更できます。すべて「関数」に属します。

- プロンプトをプリント: `fish_prompt`, `fish_right_prompt`, `fish_mode_prompt`
- コマンドが見つからない場合にどうするかを伝える: `fish_command_not_found`
- ターミナルのタイトルを変更する: `fish_title`
- fish 起動時に表示される挨拶: `fish_greeting`

デフォルトで定義されたものがありますが、自分で再定義してカスタマイズできます。例えば、`fish_greeting` を `~/.config/fish/` に `fish_greeting.fish` ファイルを作成し、以下のコードを定義することで fish 起動時に表示されるテキストを変更できます。

```shell:fish_greeting.fish
function fish_greeting
    # シェルの開始の挨拶
    echo 'Welcom to fish shell' $version
end
```

## 補助関数
以下は補助関数(helper function)として fish から提供されているコマンドです。主にプロンプトで使用するための情報を取得するものなどが含まれます。すべて「関数」に属します。

- 現在の Git や mercurial リポジトリの情報を表示: `fish_git_prompt`, `fish_hg_prompt`
- 現在の VSC(バージョンコントロールシステム)の情報を表示:  `fish_vsc_print`
- 現在の SVN リポジトリの情報を表示: `fish_svn_prompt`
- 返却されたステータスからシグナル名を与える: `fish_status_to_signal`
- 現在のディレクトリを適正にフォーマットし短縮化して表示: `prompt_pwd`, 
- ユーザーとホスト名を使用して現在のログインを記述し、chroot にいるか、ssh 経由で接続しているかを表示: `prompt_login`
- プロンプトで使用する短縮されたホスト名を表示: `prompt_hostname`
- カレントユーザーが root のような管理者かどうか確認: `fish_is_root_user`
- `$PATH` にパスを追加する: `fish_add_path`
- ラッパー関数の定義: `alias`

例えば、次のようにパスを通します。

```shell:パスをコマンドラインから通す
$ fish_add_path /opt/homebrew/bin
```

パスの通し方のパターンについては次の記事で紹介したので確認してください。

https://zenn.dev/estra/articles/zenn-fish-add-path-final-answer

## 補助コマンド
どこからでも簡単に呼び出せるよう「外部コマンド」として提供されているものが実はあります。以下の２つです。

- fish code をフォーマットする: `fish_indent`
- キー入力が生成するエスケープシーケンスを表示: `fish_key_reader`

Homebrew を使ってインストールしている場合には次のような場所に配置されています。

```shell
$ exa --tree --level=2 -a --classify /opt/homebrew/Cellar/fish/3.4.0/
/opt/homebrew/Cellar/fish/3.4.0/
├── .brew/
│  └── fish.rb
├── bin/
│  ├── fish* # fish 実体
│  ├── fish_indent* # 提供されている外部コマンド (1)
│  └── fish_key_reader* # 提供されている外部コマンド (2)
├── CHANGELOG.rst
├── COPYING
├── etc/
│  └── fish/
├── INSTALL_RECEIPT.json
├── README.rst
└── share/
   ├── doc/
   ├── fish/
   ├── man/
   └── pkgconfig/
```

# コマンドの調べ方
以上のコマンドの使い方を一気に覚えることはできないため、目についたものをその時々調べたり、やりたいことからコマンドを探していくのオススメします。

また、コマンドの使い方を調べる際の方法も紹介しておきます。以下のコマンド等を使用して調べることができます。

- `man`, `help`
- `type`
- `functions`, `bultin` `command`

基本的には `man` と `help` コマンドを調べたいコマンドの頭につけて実行することでマニュアルを確認できます。

```shell
$ man cd
$ help cd
```

実際にコマンドラインから呼び出されるものが関数・ビルトイン・外部コマンドなのかを調べるには `type` ビルトインコマンドを使用します。

```shell
# man について調べてみる
$ type man
man is a function with definition
# Defined in /opt/homebrew/Cellar/fish/3.4.0/share/fish/functions/man.fish @ line 6
function man --description 'Format and display the on-line manual pages'
    # Work around the "builtin" manpage that everything symlinks to,
    # by prepending our fish datadir to man. This also ensures that man gives fish's
    # man pages priority, without having to put fish's bin directories first in $PATH.

    # Preserve the existing MANPATH, and default to the system path (the empty string).
    set -l manpath
    if set -q MANPATH
        set manpath $MANPATH
    else if set -l p (command man -p 2>/dev/null)
        # NetBSD's man uses "-p" to print the path.
        # FreeBSD's man also has a "-p" option, but that requires an argument.
        # Other mans (men?) don't seem to have it.
        #
        # Unfortunately NetBSD prints things like "/usr/share/man/man1",
        # while not allowing them as $MANPATH components.
        # What it needs is just "/usr/share/man".
        #
        # So we strip the last component.
        # This leaves a few wrong directories, but that should be harmless.
        set manpath (string replace -r '[^/]+$' '' $p)
    else
        set manpath ''
    end
    # Notice the shadowing local exported copy of the variable.
    set -lx MANPATH $manpath

    # Prepend fish's man directory if available.
    set -l fish_manpath $__fish_data_dir/man
    if test -d $fish_manpath
        set MANPATH $fish_manpath $MANPATH
    end

    command man $argv
end
```

`type` には色々なオプションがあります。指定した名前のすべての定義を表示する `-a` オプションや関数定義などを表示しないようにする `-s` オプションを併用したり、`-t` オプションで種類だけを表示させることもできます。

```shell
# すべての定義種類を表示
$ type -sa cd
cd is a function (defined in /Users/roshi/.config/fish/functions/cd.fish)
cd is a builtin
cd is /usr/bin/cd
# 実際に呼び出されるものの種類
$ type -t cd
function
```

`functions` や `buitin`、`command` などを使用してそれぞれの種類のコマンドについて調べることも可能です。

```shell
# 定義済み関数を表示: アンダースコアから始まるやつは表示しない (省略)
$ functions -n
# すべてのビルトインコマンドを表示(省略)
$ builtin -n
# PATH 内のすべての python3 コマンドを表示
$ command -a python3
/opt/homebrew/bin/python3
/usr/bin/python3
```

