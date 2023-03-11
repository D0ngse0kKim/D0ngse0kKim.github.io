---
layout: page
title: Selenium/Introduction
description: Installation of Selenium
# img: assets/img/12.jpg
# importance: 1
category: 3rd-Party-Library
---


## Install and activate venv (optional)

```bash
% python3 -m venv .venv
% source .venv/bin/activate
(.venv) % which python3
/path/to/.venv/bin/python3
```

## Install Selenium

```bash
% python3 -m pip install -U selenium
```
`-U` (equivalant to `--upgrade`) means, "Upgrade all specified packages to the newest available version. The handling of dependencies depends on the upgrade-strategy used."

Refer [pip documentation](https://pip.pypa.io/en/stable/cli/pip_install/#cmdoption-U).

## Install Chrome

Installed chrome because I didn't use Chrome privately.

## Install Chromedriver

`webdriver_manager` described below will automatically install chromedriver.

## Install `webdriver_manager`

Official Webpage of [`webdriver_manager`](https://pypi.org/project/webdriver-manager/)

```bash
% python3 -m pip install webdriver-manager
```

### Environment Variables for configuration

Environment variables are set for configuration of `webdriver_manager`. Refer [official website of `webdriver_manager`](https://pypi.org/project/webdriver-manager/) for details of all environment variables.

* `GH_TOKEN` : personal access token on Github
* `WDM_LOCAL` : webdriver will be installed to local folder.

Generating a personal access token on Github is needed for downloading webdrivers from their official Github repository using `webdriver_manager`.

Refer [Create a personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) to make a personal access token.

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="python/3rd-party-library/selenium/introduction/gh-token.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Example of scope setting for creating a personal access token. repo:public_repo is required for accessing Github public repository."%}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="python/3rd-party-library/selenium/introduction/generated_token.png" class="img-fluid rounded z-depth-1" zoomable=true caption="token will be displayed."%}
    </div>
</div>

After creating the token, create `.env` file with the following contents:

```:.env
GH_TOKEN=<displayed token>
WDM_LOCAL=1
```

These environment variables can be imported and checked by using the following python codes:

```python
import dotenv

dotenv.load_dotenv() # returns True if .env file was loaded successfully.

import os
os.environ['GH_TOKEN']
os.environ['WDM_LOCAL'] # prints setting in .env file
```

## Links

### Tutorial

* [First Script](https://www.selenium.dev/documentation/webdriver/getting_started/first_script/)
* [Selenium Client Driver](https://www.selenium.dev/selenium/docs/api/py/index.html)
* [`selenium-python.readthedocs.io` : Locating Elements](https://selenium-python.readthedocs.io/locating-elements.html)
* [Finding web elements](https://www.selenium.dev/documentation/webdriver/elements/finders/)

### API Documentation

* [Selenium API](https://www.selenium.dev/selenium/docs/api/py/index.html)
* [Selenium with Python](https://selenium-python.readthedocs.io)

### Other

* [CSS selectors](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors).
* [XPath](https://www.w3schools.com/xml/xpath_intro.asp)

