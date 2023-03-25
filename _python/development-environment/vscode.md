---
layout: page
title: VSCode
description: extension and configuration for python development
# img: assets/img/12.jpg
# importance: 1
category: Development-Environment
---

The web page used as a reference is as follows.

* [VSCodeでの環境構築について - Qiita](https://qiita.com/nanato12/items/ddf26487eb30714251c3)
* [Getting Started with Python in VS Code](https://code.visualstudio.com/docs/python/python-tutorial)
* [Linting Python in Visual Studio Code](https://code.visualstudio.com/docs/python/linting)

## Extensions

### 1. Python
* [link](https://marketplace.visualstudio.com/items?itemName=ms-python.python)

#### extensions installed together
* [Pylance](https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance)
* [isort](https://marketplace.visualstudio.com/items?itemName=ms-python.isort)
* [Jupyter](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter)
* [Jupyter Cell Tags](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.vscode-jupyter-cell-tags)
* [Jupyter Keymap](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter-keymap)
* [Jupyter Slide Show](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.vscode-jupyter-slideshow)
* [Jupyter Notebook Renderers](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter-renderers)

### 2. Python Extension Pack
* [link](https://marketplace.visualstudio.com/items?itemName=donjayamanne.python-extension-pack)

#### extensions installed together
* [autoDocstring - Python Docstring Generator](https://marketplace.visualstudio.com/items?itemName=njpwerner.autodocstring)
* [Django](https://marketplace.visualstudio.com/items?itemName=batisteo.vscode-django)
* [IntelliCode](https://marketplace.visualstudio.com/items?itemName=VisualStudioExptTeam.vscodeintellicode)
* [IntelliCode API Usage Examples](https://marketplace.visualstudio.com/items?itemName=VisualStudioExptTeam.intellicode-api-usage-examples)
* [Jinja](https://marketplace.visualstudio.com/items?itemName=wholroyd.jinja)
* [Python Environment Manager](https://marketplace.visualstudio.com/items?itemName=donjayamanne.python-environment-manager)
* [Python Indent](https://marketplace.visualstudio.com/items?itemName=KevinRose.vsc-python-indent)

### 3. Error Lens
[Error Lens](https://marketplace.visualstudio.com/items?itemName=usernamehw.errorlens)

### 4. indent-rainbow
[indent-rainbow](https://marketplace.visualstudio.com/items?itemName=oderwat.indent-rainbow)

### 5. Trailing Space
[Trailing Spaces](https://marketplace.visualstudio.com/items?itemName=shardulm94.trailing-spaces)

<!---
Query:Name: (.*)$(\n|[^\n])*?Link: (.*)$
Replacement:[$1]($3)
でreplacementすると、コピーしたextension情報をリンクに変換可能
--->

## `settings.json`

### Trim trailing spaces
```json
    "files.trimTrailingWhitespace": true
```

## Tools

### 1. `black`
* code formatter for python
* CLI options can be found from [here](https://black.readthedocs.io/en/stable/usage_and_configuration/the_basics.html)
* [comparison of autopep8 vs black](https://medium.com/mlearning-ai/python-auto-formatter-autopep8-vs-black-and-some-practical-tips-e71adb24aee1)

#### installation and usage
1. install `black` from `pip`
    ```sh
    % python3 -m pip install black
    % black test.py
    ```
2. find path of `black` using `which` or `where` command
    ```sh
    % where black
    /.venv/bin/black
    ```
3. add options written in below to `settings.json`
    ```json
    "editor.formatOnSave": true,
    "python.formatting.blackPath": "${workspaceFolder}/.venv/bin/black",
    "python.formatting.blackArgs": ["--line-length=79", ],
    "python.formatting.provider": "black",
    ```

### 2. `flake8`
* style guide tool for python source code
* CLI options for `flake8` can be found from [here](https://flake8.pycqa.org/en/latest/user/options.html)

#### installation and usage
1. install `flake8` from `pip`
    ```sh
    % python3 -m pip install flake8
    % flake8 test.py
    ```
2. find path of `flake8` using `which` or `where` command
    ```sh
    % which flake8
    /.venv/bin/flake8
    ```
3. add options written in below to `settings.json`
    ```json
    "python.linting.flake8Enabled": true,
    "python.linting.flake8Path": "${workspaceFolder}/.venv/bin/flake8",
    "python.linting.flake8CategorySeverity.E": "Warning",
    "python.linting.flake8CategorySeverity.F": "Warning",
    "python.linting.flake8CategorySeverity.W": "Warning",
    ```

### 3. `isort`
* sorting tool for `import` statement

#### installation and usage
1. install `isort` from `pip`
    ```sh
    % python3 -m pip install isort
    % isort test.py
    ```
2. find path of `isort` using `which` or `where` command
    ```sh
    % which isort
    /.venv/bin/isort
    ```
3. add options written in below to `settings.json`
    ```json
    "isort.path": ["${workspaceFolder}/.venv/bin/isort",],
    "editor.codeActionsOnSave": {
        "source.organizeImports": true
    },
    ```

### 4. `mypy`
* type checker tools
* several options for `mypy` can be found in [here](https://mypy.readthedocs.io/en/stable/running_mypy.html)

#### installation and usage
1. install `mypy` from `pip`
    ```sh
    % python3 -m pip install mypy
    % mypy test.py
    ```
2. find path of `mypy` using `which` or `where` command
    ```sh
    % which mypy
    /.venv/bin/mypy
    ```
3. add options written in below to `settings.json`
    ```json
    "python.linting.mypyEnabled": true,
    "python.linting.mypyPath": "${workspaceFolder}/.venv/bin/mypy",
    "python.linting.mypyCategorySeverity.error": "Information",
    "python.linting.mypyCategorySeverity.note": "Information",
    "python.linting.mypyArgs": [
        "--warn-return-any",        // function returns Any will be warned
        "--disallow-untyped-defs",  // function with untyped return value cannot be allowed
        "--no-implicit-optional",   // None cannot be assigned to type annotated variable
        "--disallow-untyped-calls", // function does not have return type cannot be called
        //"--ignore-missing-imports", // imported library will not be checked
        "--follow-imports=normal",  // follows all imports and run type check
        "--show-column-numbers",
        "--no-pretty"
    ],
    ```
