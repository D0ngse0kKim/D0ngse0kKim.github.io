---
layout: page
title: Tutorial
description: Tutorial of Jekyll
# img: assets/img/12.jpg
# importance: 1
category: Jekyll
---

## Installation

### References

* [Installation](https://jekyllrb.com/docs/installation/)
* [Jekyll on macOS](https://jekyllrb.com/docs/installation/macos/)
* [Step by Step Tutorial](https://jekyllrb.com/docs/step-by-step/01-setup/)

### Install `ruby`

install `ruby` and `chruby`(Version manaber of `ruby`) from `brew`

```bash
% brew install chruby ruby-install xz
% ruby-install ruby 3.1.3
% echo "source $(brew --prefix)/opt/chruby/share/chruby/chruby.sh" >> ~/.zshrc
% echo "source $(brew --prefix)/opt/chruby/share/chruby/auto.sh" >> ~/.zshrc
% echo "chruby ruby-3.1.3" >> ~/.zshrc # run 'chruby' to see actual version
% ruby -v
ruby 3.1.3p185 (2022-11-24 revision 1a6b16756e) [arm64-darwin21]
```

### Install Jekyll

```bash
% gem install jekyll bundler
```

### Write dependencies

Create a new `Gemfile` into a project folder to list dependencies of the project.

```bash:console
% cd /path/to/project/folder
% bundle init
```

Edit `Gemfile` with text editor and add jekyll as a dependency

```bash:console
% code Gemfile
```

```:Gemfile
gem "jekyll"
```

## Create a site

Create `index.html` in **root** of your project folder with the following contents:

```html:index.html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Home</title>
  </head>
  <body>
    <h1>Hello World!</h1>
  </body>
</html>
```

## Build

* `jekyll build` - Builds the site and outputs a static site to a directory called `_site`.
* `jekyll serve` - Does jekyll build and runs it on a local web server at `http://localhost:4000`, rebuilding the site any time you make a change.

## Liquid

### Objects

* Objects can be used to print predefined variables as content on a page.
* Use double curly braces for object.
    * `{{ page.title }}` displays `page.title` variable.
* [Variables](https://jekyllrb.com/docs/variables/) lists available data on jekyll.

### Tags

{% raw %}

* Tags defines the logic and control flow for templates.
* Use curly braces and percent sign for tags: `{%` and `%}`
* `if` statement in jekyll can be used inside of Tags
* An example below displays sidevar contents if `page.show_sidebar` is true.

    ```html
    {% if page.show_sidebar %}
    <div class="sidebar">
        sidebar content
    </div>
    {% endif %}
    ```

{% endraw %}

### Filters

{% raw %}

* Filters change the output of a Liquid object.
* They are used within an output and are separated by a `|`.
* An example below displays capitalized "HI" instead of "hi".

    ```
    {{ "hi" | capitalize }}
    ```

{% endraw %}

### Use Liquid

{% raw %}

An example below displays downcased `hello world!` on `index.html`.

A front matter is needed to process Liquid on Jekyll.

```html:index.html
---
---

<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Home</title>
  </head>
  <body>
    <h1>{{ "Hello World!" | downcase }}</h1>
  </body>
</html>
```

{% endraw %}

## Front Matter

### Front Matter

{% raw %}

A front matter is a snippet of YAML placed between two triple-dashed lines at the start of a file.

Variables defined inside of front matter can be accessed using `page.var_inside_of_front_matter`.

An example below displays `title` inside of front matter on the page title of web browser.

```html:index.html
---
title: Home
---
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>{{ page.title }}</title>
  </head>
  <body>
    <h1>{{ "Hello World!" | downcase }}</h1>
  </body>
</html>
```
{% endraw %}

## Layouts

Layouts in Jekyll can be used for providing template layout to each html or markdown files.

### Creating a layout

{% raw %}

Layout files is stored in a directory called `_layouts`.

An example below defines page title and contents of each page files.

A special variable `contents` is used for rendering of content of the each pages.

```html:_layouts/default.html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>{{ page.title }}</title>
  </head>
  <body>
    {{ content }}
  </body>
</html>
```

{% endraw %}

### Use layouts

#### html

{% raw %}

`index.html` can be re-written to simple contents by using `_layouts/default.html`.

```html:index.html
---
layout: default
title: Home
---
<h1>{{ "Hello World!" | downcase }}</h1>
```

{% endraw %}


#### Markdown

{% raw %}

`about.md` is a markdown file that uses `_layouts/default.html` file to render its contents.

```markdown:about.md
---
layout: default
title: About
---
# About page

This page tells you a little bit about me.
```

An webpages generated from `about.md` can be accessed from a URL `http://localhost:4000/about.html`.

{% endraw %}

## Includes

`include` tag can be used to include content from another file stored in an `_includes` folder.

### Include usage

{% raw %}

An example below defines an navigation file in the `_includes` folder.

This file renders links to each pages with red highlighting to current page link.

```html:_includes/navigation.html
<nav>
    <a href="/" {% if page.url == "/" %}style="color: red;"{% endif %}>
        Home
    </a>
    <a href="/about.html" {% if page.url == "/about.html" %}style="color: red;"{% endif %}>
        About
    </a>
</nav>
```

`_includes/navigation.html` file can be embedded to another html file using `include` tag.

```html:_layouts/default.html
<!doctype html>
<html>
    <head>
        <meta charset="utf-8">
        <title>{{ page.title }}</title>
    </head>
    <body>
        {% include navigation.html %}
        {{ content }}
    </body>
</html>
```

{% endraw %}

## Data files

Jekyll supports loading data from YAML, JSON, and CSV files located in a `_data` directory.

### Data file usage

{% raw %}

`_data/navigation.yml` below defines data of url and name of each pages.

```yaml:_data/navigation.yml
- name: Home
  link: /
- name: About
  link: /about.html
```

Data in this file can be accessed by using `site.data.navigation`.

```html:_includes/navigation.html
<nav>
    {% for item in site.data.navigation %}
        <a href="{{ item.link }}" {% if page.url == item.link %}style="color: red;"{% endif %}>
        {{ item.name }}
        </a>
    {% endfor %}
</nav>
```

{% endraw %}

