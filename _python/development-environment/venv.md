---
layout: page
title: venv
description: virtual environment in development of python
# img: assets/img/12.jpg
# importance: 1
category: Development-Environment
---

## Creating virtual environments

The following commands creates a folder for building a virtual environment in the `.venv` folder. [^mflag]

```
python3 -m venv .venv
```

[^mflag]:Details of a `-m` flag can be found in [here](https://docs.python.org/3/using/cmdline.html).

## Activate/deactivate virtual environment

| Commands                    | Details                                    |
| :-------------------------- | :----------------------------------------- |
| `source .venv/bin/activate` | activates virtual environment[^howtocheck] |
| `deactivate`                | deactivate virtual environment             |

[^howtocheck]: It can be confirmed that the path of `python3` is the path to the folder in the `/.venv/bin/python3` by using `which python` or `where python3`.


## Install packages

Packages can be installed by using [pip](../packaging/pip.md).

## References

* [`venv` - Creation of virtual environments](https://docs.python.org/3/library/venv.html)
* [Installing packages using pip and virtual environments](https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/#creating-a-virtual-environment)
