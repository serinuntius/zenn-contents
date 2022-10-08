---
title: "環境を準備する"
---


## Rustのインストール
```bash
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# .bashrc, .zshrcに追記する
export PATH="$HOME/.cargo/bin:$PATH"

# シェル再起動
$ exec $SHELL -l

$ rustup --version
rustup 1.25.1 (bb60b1e89 2022-07-12)
info: This is the version for the rustup toolchain manager, not the rustc compiler.
info: The currently active `rustc` version is `rustc 1.66.0-nightly (f5193a9fc 2022-09-25)`

# とりあえずstableをインストール
$ rustup install stable

```

## ツールのインストール
```
$ cargo install store_daemon --all-features --version "0.8.0"

$ cargo install rgb-std
$ cargo install rgb20 --all-features --version 0.8.0-rc.4git clone git@github.com:RGB-WG/rgb-node.git
$ cd rgb-node
$ cargo install --all-features --path .
$ cargo install --all-features --path cli

$ exec $SHELL -l
```



