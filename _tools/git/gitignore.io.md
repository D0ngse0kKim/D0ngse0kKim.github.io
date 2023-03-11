---
layout: page
title: gitignore.io
description: .gitignore generator
# img: assets/img/12.jpg
# importance: 1
category: Git
---


## Adding `gi` command to shell

gitignore.io can be used by `curl` command to API page[^APIPage] of gitignore.io.

A script in below adds `gi` command to zsh which retrieves template of `.gitignore`.

```bash
echo "function gi() { curl -sLw "\n" https://www.toptal.com/developers/gitignore/api/\$@ ;}" >> \
~/.zshrc && source ~/.zshrc
```

Refer this webpage([Install - command line](https://docs.gitignore.io/install/command-line)) for details.

[^APIPage]:[https://www.toptal.com/developers/gitignore/api/]()

### Usage

The following `gi` command with `windows,macos,python` gets template of `.gitignore` for windows, macOS, python development environment.

```
gi windows,macos,python
```

## Install `gi` extension to VSCode

`gi` extension can be found here

* [install - editor extensions](https://docs.gitignore.io/install/editor-extensions)
* [Marketplace Page](https://marketplace.visualstudio.com/items?itemName=rubbersheep.gi)

### Usage

`gi` command can be found in command palette

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="tools/git/gitignore.io_command-palette.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
