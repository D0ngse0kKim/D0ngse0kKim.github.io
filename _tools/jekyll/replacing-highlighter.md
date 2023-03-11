---
layout: distill
title: Replacing Highlighter
description: How to replace highlighter in jekyll
# img: assets/img/12.jpg
# importance: 1
category: Jekyll

toc:
  - name: Abstract
  - name: Disable Jekyll's default syntax highlighter Rouge
  - name: Use highlight.js as highlighter
  - name: Add line numbers display setting
  - name: Display file name of each code block
  - name: Injection of external code from github
  - name: Commit
  - name: References
---

## Abstract

This is summary of the way to replace Jekyll's default highlighter Rouge to [highlight.js](https://highlightjs.org).

## Disable Jekyll's default syntax highlighter Rouge

Disabling Jekyll's default syntax highlighter is required for using [highlight.js](https://highlightjs.org) as syntax highlighter.

Default settings of [al-folio](https://github.com/alshedivat/al-folio), which I uses as a template for github-pages used [Rouge](https://github.com/rouge-ruby/rouge) as syntax highlighter.

The following is the configuration in _config.yml when Rouge was used as syntax highlighter.

```yaml:_config.yml
# Markdown and syntax highlight
markdown: kramdown
highlighter: rouge
kramdown:
  input: GFM
  syntax_highlighter_opts:
    css_class: 'highlight'
    span:
      line_numbers: false
    block:
      line_numbers: false
      start_line: 1
```

The default syntax highlighter can be disabled by changing setting as follows.

```yaml:_config.yml
markdown: kramdown
highlighter: none
kramdown:
  input: GFM
  syntax_highlighter_opts:
    disable: true
```

## Use highlight.js as highlighter

`_includes/header.html` was embedded for every pages genetared by Jekyll in [al-folio](https://github.com/alshedivat/al-folio) template which I uses for github-pages.

So I added the following codes in `_includes/header.html` for using [highlight.js](https://highlightjs.org) as syntax highlighter.

```html:_includes/header.html
<link rel="stylesheet" href="//cdn.jsdelivr.net/gh/highlightjs/cdn-release@11.7.0/build/styles/default.min.css">
<script src="//cdn.jsdelivr.net/gh/highlightjs/cdn-release@11.7.0/build/highlight.min.js"></script>
<script>hljs.highlightAll();</script>
```

This codes can be found in [How to use highlight.js](https://highlightjs.org/usage/).

Codes are highlighted by adding `<language>` specifier after three backticks of a code block.
The following example shows a code block that uses syntax highlighter for python.

````markdown
```python
def printarguments(*args):
    print(args)
```
````

## Add line numbers display setting

Line numbers can be displayed by using [highlight.js-line-numbers.js](https://github.com/wcoder/highlightjs-line-numbers.js/).

First, I inserted the following codes into `_includes/header.html` for enabling line numbers display.

```html:_includes/header.html
<script src="//cdn.jsdelivr.net/npm/highlightjs-line-numbers.js@2.8.0/dist/highlightjs-line-numbers.min.js"></script>
<script>hljs.initLineNumbersOnLoad();</script>
```

In order to adjust line numbers related display, I have added the following css settings that can be found [here](https://emmalanglab.com/highlightjs-line-numbers-ja/).

```css:_sass/_base.scss
.hljs-ln {
  // border display disabled
  margin-bottom: 0;
  border: none;
}

.hljs-ln tr, .hljs-ln td {
  // border display disabled
  border: none;
}

.hljs-ln-numbers > div {
  font-family: inherit;
  -webkit-touch-callout: none; /* Hiding of pop-up menus that are displayed by long-pressing on phones, etc. */
  -webkit-user-select: none;   /* Allow line numbers to be excluded when copying code */
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  text-align: right;
  color: var(--global-theme-color); /* color of characters of line numbers */
  opacity: 0.65;
  vertical-align: top;
  padding-right: 10px;
  width: 2rem; /* width of line numbers */
  min-width: 2rem; /* mininum width for line numbers display */
  font-size: 80%;
  line-height: 1.55em;
}

.hljs-ln-code.hljs-ln-line {
  padding-left: 8px;
  font-size: 87.5%;
  line-height: 1.55em;
  border-left: 1px solid var(--global-theme-color);
}
```

`var(--global-theme-color)` must be replaced with the value you wish to use as the color of the line number characters to be displayed.

In [al-folio](https://github.com/alshedivat/al-folio), the following settings are set for default color `#B509AC`.

```css:_variables.scss
$purple-color:        #B509AC !default;
```

```css:_theme.scss
:root {
  --global-theme-color: #{$purple-color};
}
```

## Display file name of each code block

File name can be specified by using specifier like `<language>:<filename>` after three backticks of a code block.

The following code block specifies syntax highlight for python language and `filename.py` as file name.

````markdown
```python:filename.py
def printarguments(*args):
    print(args)
```
````

This displays code block like this.

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="tools/jekyll/replacing-highlighter/code-block-with-file-name.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Example of a code block displaying a file name"%}
    </div>
</div>

### Javascript code for inserting elements which displays file name of code block into code block tags.

To do this, add a javascript code which parses language and file name specifier of code block.

```javascript:highlight.html
<scripts>
    const code = document.getElementsByTagName('code');
    Array.from(code).forEach(append_code_block_header);

    function append_code_block_header(elemCode){
        if(!elemCode.className){return;}else{}

        const [language, urlCodeBlock] = get_language_and_url(elemCode);
        if(!urlCodeBlock){return;}else{}

        replace_className(elemCode, language);
        elemCodeBlockHeader = create_elem_code_block_header(urlCodeBlock);
        append_elem_code_block_header_to_previous(elemCode, elemCodeBlockHeader);
    }

    function get_language_and_url(elemCode){
        return elemCode.className.split(/:(.*)/s)
    }

    function replace_className(elemCode, newName){
        elemCode.classList.remove(elemCode.className);
        elemCode.classList.add(newName);
    }

    function create_elem_code_block_header(urlCodeBlock){
        var elemCodeBlockHeader = document.createElement('a');
        elemCodeBlockHeader.textContent = urlCodeBlock;
        if (
            (urlCodeBlock.startsWith('http://'))
            || (urlCodeBlock.startsWith('https://'))
        ){
            elemCodeBlockHeader.setAttribute('href', urlCodeBlock);
            elemCodeBlockHeader.setAttribute('target', '_blank');
        }
        elemCodeBlockHeader.className = 'language-plaintext';

        return elemCodeBlockHeader;
    }

    function append_elem_code_block_header_to_previous(elemCode, elemToAppend){
        var elemPre = elemCode.parentElement;
        elemPre.insertBefore(elemToAppend, elemPre.firstChild);
        elemPre.classList.add('code-block-header');
    }
</scripts>
```

This code performs the following processing.
1. Find `<pre><code> code block contents </code></code>` tags.
2. Split language and file name specifier in `<code class='language:filename'>` into `language` and `filename`. (`get_language_and_url()`)
3. Replace class name from `<pre><code class='language:filename'>` to `<pre><code class='language'>`. (`replace_className()`)
4. Generate `<a>filename</a>` tags from file name specifier.
    * If `filename` is a url starts with `http://` or `https://`, adds a hyperlink to the url. (`create_elem_code_block_header()`)
5. Insert the `<a></a>` tag in front of `<code class='language'>` tag. (`append_elem_code_block_header_to_previous()`)
6. Insert class name into `pre` tag like `<pre class='code-block-header'>`. (`append_elem_code_block_header_to_previous()`)

### Styling of code block filename display

```css:_sass/_base.scss
pre.code-block-header {
  position: relative;
  padding-top: 24px;
  a:first-child {
    background: var(--global-code-bg-color-darker);
    color: var(--global-theme-color);
    display: block;
    font-size: 80%;
    font-weight: 700;
    padding: 1px 10px;
    position: absolute;
    top: 0;
    right: 0;
    z-index: 10;
    opacity: 2;
    border-bottom: 0;

    /* Maximum width of 40%, word wrapping if exceeded */
    max-width: 40%;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
}
```

## Injection of external code from github

Add a function that allows you to display code on Github as a code block on your own page by using the following `<div>` tag.
```html
<div class='embed-github-src' repo='fluentpython/example-code-2e' branch='master' path='07-1class-func/tagger.py' line=1-23 language='python'></div>
```

* `class` : Mandatory, specify `'embed-github-src'`.
* `repo` : Mandatory, repository name, for example `'fluentpython/example-code-2e'`
* `branch` : Mandatory, branch name, for example `'master'`
* `path` : Mandatory, file path on repository, `'07-1class-func/tagger.py'`
* `language` : Mandatory, language for syntax highlighter, for example `'python'`
* `line` : Optional, `<startline>-<endline>`

### Javascript code

```javascript:highlight.html
/*
Parse the following format of tag div.
<div class='embed-github-src'               : Mandatory, 'embed-github-src'
     repo='fluentpython/example-code-2e'    : Mandatory, repository name
     branch='master'                        : Mandatory, branch of repository
     path='07-1class-func/tagger.py'        : Mandatory, path of file
     line=1-23                              : Optional, line of file
     language='python'
></div>
*/
var divs = document.getElementsByClassName('embed-github-src');
Array.prototype.forEach.call(divs, embed_external_source);

function embed_external_source(elemDiv) {
    var url_raw = get_url_raw(elemDiv);
    var url_blob = get_url_blob(elemDiv);
    var language = elemDiv.getAttribute('language');
    var lines = get_lines(elemDiv);
    var lineStart = lines[0];
    var lineEnd = lines[1];

    fetch(url_raw).then(function(res) {
        return res.text();
    }).then(function(out) {
        var elemCode = create_elem_code_block(language, url_blob, out, lineStart, lineEnd);
        var elemPre = create_elem_pre_from_code_block(elemCode);
        elemDiv.appendChild(elemPre);
        hljs.highlightBlock(elemCode);
        hljs.lineNumbersBlock(elemCode);
    }).catch(function(err) {
        throw err;
    });
}

function get_url_raw(elemDiv){
    const baseurl = 'https://raw.githubusercontent.com';
    const repo = elemDiv.getAttribute('repo');
    const branch = elemDiv.getAttribute('branch');
    const path = elemDiv.getAttribute('path');

    return [baseurl, repo, branch, path].join('/');
}

function get_url_blob(elemDiv){
    const baseurl = 'https://github.com';
    const repo = elemDiv.getAttribute('repo');
    const branch = elemDiv.getAttribute('branch');
    const path = elemDiv.getAttribute('path') + get_line_specifier(elemDiv);

    return [baseurl, repo, 'blob', branch, path].join('/');
}

function get_lines(elemDiv){
    const attributeLine = elemDiv.getAttribute('line');
    if(!attributeLine){
        return [0, 0];
    }else{}

    var [lineStart, lineEnd] = attributeLine.split(/-(.*)/s);
    lineStart = parseInt(lineStart);
    lineEnd = parseInt(lineEnd);

    if(isNaN(lineStart)){
        lineStart = 0;
    }else{}

    if(isNaN(lineEnd)){
        lineEnd = 0;
    }else{}

    return [lineStart, lineEnd];
}

function get_line_specifier(elemDiv){
    const lines = get_lines(elemDiv);

    if((lines[0] == 0) && (lines[1] == 0)){
        return '';
    }else{}

    if(lines[1] == 0){
        lines[1] == 99999999;
    }else{}

    return '#L' + String(lines[0]) + '-L' + String(lines[1]);
}

function create_elem_code_block(language, url, contents, lineStart, lineEnd){
    var elemCode = document.createElement('code');
    elemCode.className = 'language-' + language + ':' + url;
    elemCode.textContent = slice_by_line_number(contents, lineStart, lineEnd);

    return elemCode;
}

function slice_by_line_number(context, lineStart, lineEnd){
    if((lineStart == 0) && (lineEnd == 0)){
        return context;
    }else{}

    if(lineStart == 0){
        lineStart = 1;
    }else{}

    if(lineEnd == 0){
        lineEnd == 99999999;
    }else{}

    return context.split('\n').slice(lineStart-1, lineEnd).join('\n');
}

function create_elem_pre_from_code_block(elemCode){
    var elemPre = document.createElement('pre');
    elemPre.appendChild(elemCode);
    append_code_block_header(elemCode);

    return elemPre;
}
```

## Commit

Refer this [Commit](https://github.com/D0ngse0kKim/D0ngse0kKim.github.io/commit/f587b3d0e1c3703b8b00cfa66e238c9698d1b89d) of actual code changes for this functions.

## References

* [Disable Jekyll's default syntax highlighter Rouge](https://vilcins.medium.com/disable-jekylls-default-syntax-highlighter-rouge-12130ccac779)
* [Add file name to code block in Jekyll on Github pages](https://hachy.github.io/2018/11/14/add-file-name-to-code-block-in-jekyll-on-github-pages.html)
* [Jekyll - Highlighting](https://jojozhuang.github.io/tutorial/jekyll-highlighting/)
* [highlight.jsに行番号を追加する方法](https://emmalanglab.com/highlightjs-line-numbers-ja/)
* [highlightjs-line-numbers.js](https://github.com/wcoder/highlightjs-line-numbers.js/)
* [How to Embed Raw Code From Github in Ghost Posts](https://www.zuar.com/blog/ghost-embed-raw-code-from-github/)
