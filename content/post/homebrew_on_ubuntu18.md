---
title: "HomebrewをUbuntu18.04にインストール"
date: 2020-10-04T12:07:18+09:00
lastmod: 2020-10-04T12:07:18+09:00
draft: false
description: ""
tags: [Homebrew]

---

aptとかsnapと何が違うのか理解してないけど、Hugoをインストールするために導入した。  
Home下とかにインストールできるらしい (sudoがいらない)。

# Install
普段fishを使ってるので、bashに切り替えてから公式にかかれているコマンドを実行した。  
sudo権限が必要なディレクトリにインストールするか、Home下の`~/.linuxbrew`にインストールするか選べる。
```bash
bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
# home下にInstallしたいので、ctrl-Dを選んだ
```

# path設定
```bash
# 一時的にPathを設定する場合
eval $(~/.linuxbrew/bin/brew shellenv)

# bash起動時に設定されるようにする場合
echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.profile

# fishを使っている場合
# 公式の bin/brew shellenv だと他の変数も設定しているが、必要なさそうなのでbinだけPathに追加しておく
echo "set -x PATH ~/.linuxbrew/bin $PATH" >>~/.config/fish/config.fish
```

## 参考
- https://brew.sh/
- https://docs.brew.sh/Homebrew-on-Linux
- http://kikkii.hatenablog.com/entry/2017/12/19/121605
