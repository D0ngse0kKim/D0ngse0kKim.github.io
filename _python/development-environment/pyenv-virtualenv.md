---
layout: page
title: pyenv-virtualenv
description: python version manager
# img: assets/img/12.jpg
# importance: 1
category: Development-Environment
---

参考にしたページ
* [こちら](https://qiita.com/ms-rock/items/72b8f1abc661c539bb09)
* [こちら](https://qiita.com/ksato9700/items/5d9eba10fe6b8e064178)

## 概要
* pythonのバージョンを自由に変えられるツール
* 特定のバージョンを導入して、名前をつけてそれぞれ管理できる
    * `venv`的な`pyenv`

## 導入
### macOS
1. `brew install pyenv-virtualenv`
2. `code ~/.zshrc`して以下を追加 (パスを通して`pyenv`コマンドを追加)
    ```bash
    export PYENV_ROOT=${HOME}/.pyenv
    if [ -d "${PYENV_ROOT}" ]; then
        export PATH=${PYENV_ROOT}/bin:$PATH
        eval "$(pyenv init -)"
        eval "$(pyenv virtualenv-init -)" # <- この行を追加
    fi
    ```

## 使い方
### 既存のインストールしたバージョンから環境を新たに作成
```bash
% pyenv virtualenv 3.11.1 envLint
% pyenv versions
  system
* 3.11.0 (set by PYENV_VERSION environment variable)
  3.11.1
  3.11.1/envs/envLint       # <- この二つが生成される
  envLint                   # <- この二つが生成される
```

### 追加した環境を削除
```bash
% pyenv uninstall envLint
pyenv: remove /Users/dongseokkimp/.pyenv/versions/envLint? [y|N] y
pyenv-virtualenv: remove /Users/dongseokkimp/.pyenv/versions/3.11.1/envs/envLint? (y/N) y
% pyenv versions
  system
  3.11.0
  3.11.1
```
