---
title: "はてなブログからHugoに移行した"
date: 2020-10-04T11:42:39+09:00
lastmod: 2020-10-04T11:42:39+09:00
draft: false
keywords: []
description: ""
tags: [Hugo]
categories: []
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams:
  enable: false
  options: ""

---

なんとなく、はてなブログからHugoに移行した

# 環境構築 on Ubuntu18.04
## Install
Hugoのインストールは、公式を見るとhomebrewを推奨してるっぽい  
(aptにもあるが、バージョンが古い？)
Homebrewがインストールされていない場合は、まず [Homebrewをインストール]({{< ref "homebrew_on_ubuntu18.md" >}}) する必要がある。
```bash
brew install hugo
hugo version
# Hugo Static Site Generator v0.75.1/extended linux/amd64 BuildDate: unknown
```

## サイトの作成
カレントディレクトリの下に[site_name]というディレクトリが作成される
```bash
hugo new site [site_name]
# gitで管理する場合
cd [site_name]
git init
```

## テーマの設定
https://themes.gohugo.io から適当に良さそうなテーマを探して使う
```bash
git submodule add https://github.com/olOwOlo/hugo-theme-even themes/even
echo 'theme = "even"' >> config.toml
# テーマの設定ファイルをコピー
cd [site_name]
cp themes/even/exampleSite/config.toml config.toml
# 適当にconfig.tomlのautherなどを書き換え
# evenのテーマだと、タグとカテゴリーが設定できるが片方だけでいいので、カテゴリーは削除した
# defaultContentLanguageをjaに設定
```

## 記事の作成
```bash
# content/<CATEGORY>/<FILE>.<FORMAT> に追加される
hugo new posts/my-first-post.md
# evenのテーマだとpostsではなくpostを使わないとだめらしい
hugo new post/my-first-post.md
```

## ローカルでプレビュー
```bash
hugo server -D
# -Dオプションで下書きも確認できる
# http://localhost:1313/ で確認できる
```

## HTML生成
```bash
hugo -D # ./public/ に生成される
```

## 参考
- https://gohugo.io/getting-started/installing
- https://gohugo.io/getting-started/quick-start/
- https://knowledge.sakura.ad.jp/22908/



