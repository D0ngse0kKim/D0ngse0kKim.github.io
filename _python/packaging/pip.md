---
layout: page
title: pip
description: package installer in python
# img: assets/img/12.jpg
# importance: 1
category: Packaging
---

## References

[Installing packages using pip and virtual environments](https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/#creating-a-virtual-environment)

[`venv`](venv.md)導入をおすすめ

## Install Packages

Packages can be installed to your environment using the commands described below.

| Example of command                              | Meaning                                                    |
| :---------------------------------------------- | :--------------------------------------------------------- |
| `python3 -m pip install requests`               | install `requestes` package                                |
| `python3 -m pip install requests==2.18.4`       | install `requestes` package of version 2.18.4              |
| `python3 -m pip install requests>=2.0.0,<3.0.0` | install the latest version of 2.x                          |
| `python3 -m pip install --pre requests`         | install pre-release versions                               |
| `python3 -m pip install requests[security]`     | install with extras `security` which specified in brackets |

## Install from source

It is possible to install packages from source.

```
cd google-auth
python3 -m pip install .
```

The changes to the source directory will immediately affect the installed package without re-install the package.

```
python3 -m pip install --editable .
```

## Install from local archives

Package can be installed from a Distribution Package's archive file saved in a local folder.
(zip, wheel, or tar file can be used to install packages)

```
python3 -m pip install requests-2.18.4.tar.gz
```

It is possible to install packages using files on a system with limited connectivity
without using [PyPI](https://packaging.python.org/en/latest/glossary/#term-Python-Package-Index-PyPI).

```
python3 -m pip install --no-index --find-links=/local/dir/ requests
```

## Upgrading package

Packages can be upgraded using the following commands:

```
python3 -m pip install --upgrade requests
```

## Install with requirement files

The format of the requirement file `requirements.txt` is, for example:

```
requests==2.18.4
google-auth==1.1.0
```

Packages can be installed using `requirements.txt` with the `-r` option.

```
python3 -m pip install -r requirements.txt
```

## Freezing dependencies

A `freeze` command can display the currently installed packages.

```
python3 -m pip freeze
```

Here is an example of the output of a `freeze` command.
The output of the `freeze` command can be used as a `requirements.txt` file for `-r` flag of `pip install` command.

```
attrs==22.2.0
contourpy==1.0.6
cycler==0.11.0
fonttools==4.38.0
iniconfig==2.0.0
kiwisolver==1.4.4
matplotlib==3.6.2
mypy==0.991
mypy-extensions==0.4.3
numpy==1.23.5
packaging==22.0
Pillow==9.3.0
pluggy==1.0.0
pyparsing==3.0.9
pytest==7.2.1
python-dateutil==2.8.2
six==1.16.0
typing_extensions==4.4.0
```
