---
layout: page
title: pyenv
description: python version manager
# img: assets/img/12.jpg
# importance: 1
category: Development-Environment
---

参考にしたページは[こちら](https://qiita.com/ms-rock/items/72b8f1abc661c539bb09)

## 概要
pythonのバージョンを自由に変えられるツール

## 導入
### macOS
1. `brew install pyenv`
2. `code ~/.zshrc`して以下を追加 (パスを通して`pyenv`コマンドを追加)

    ```bash
    export PYENV_ROOT=${HOME}/.pyenv
    if [ -d "${PYENV_ROOT}" ]; then
        export PATH=${PYENV_ROOT}/bin:$PATH
        eval "$(pyenv init -)"
    fi
    ```

## 使い方
### インストール可能なバージョンの確認
* `pyenv install -l`
    * 数が多いので、`3.**.**`を調べるために`pyenv install -l | grep -E '^\s*3\.'`で検索した

### インストール
* `pyenv install 3.11.0`

### インストールされたバージョンのチェック
* `pyenv versions`

    ```
    ~/Desktop [23:20:01] % pyenv versions
    * system (set by /Users/dongseokkimp/.pyenv/version)
      3.11.0
    ```

### 使うバージョンの選択
* `pyenv global 3.11.0`

    ```
    ~/Desktop [23:20:07] % pyenv global 3.11.0
    ~/Desktop [23:21:03] % pyenv versions
      system
    * 3.11.0 (set by /Users/dongseokkimp/.pyenv/version)
    ```

* `3.11.0`の前に`*`がついていることを確認
* `pyenv local 3.11.0`のように、`local`を使ってのしても可能
    * この場合、上記コマンドを打った当時のフォルダにのみ適用される

### 特定のバージョンで仮想環境を作りたい
[`pyenv-virtualenv`](pyenv-virtualenv.md)をご参照
