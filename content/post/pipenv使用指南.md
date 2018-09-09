---
title: pipenv使用指南
date: 2018-01-08T14:16:29+08:00
tags:
    - pipenv
categories:
    - python库相关
slug: pipenv-tour
description: >
    本文主要介绍了python包管理工具: pipenv的使用方法。
---

[`pipenv`](https://github.com/pypa/pipenv)是[Python官方推荐](https://packaging.python.org/tutorials/managing-dependencies/#managing-dependencies)的包管理工具。可以说，它集成了`virtualenv`, `pip`和`pyenv`三者的功能。其目的旨在集合了所有的包管理工具的长处，如: `npm`, `yarn`, `composer`等的优点。

它能够自动为项目创建和管理虚拟环境，从`Pipfile`文件添加或删除安装的包，同时生成`Pipfile.lock`来锁定安装包的版本和依赖信息，避免构建错误。


`pipenv`主要解决了如下问题:

* 不用再单独使用`pip`和`virtualenv`, 现在它们合并在一起了
* 不用再维护`requirements.txt`, 使用`Pipfile`和`Pipfile.lock`来代替
* 可以使用多个python版本(`python2`和`python3`)
* 在安装了`pyenv`的条件下，可以自动安装需要的Python版本

## 安装

为了方便使用, 建议**全局**安装

```bash
$ pip install pipenv
```

<!--more-->

## 基本概念

* 虚拟环境如果不存在的话，会自动创建
* 当`install`命令没有传递参数指定安装包时，所有`[packages]`里指定的包都会被安装
* `pipenv --three`可以初始化一个`python3`版本的虚拟环境
* `pipenv --two`可以初始化一个`python2`版本的虚拟环境

## 添加shell补齐

如果使用的是`bash`, 可添加下面语句到`.bashrc`或`.bash_profile`

```bash
eval "$(pipenv --completion)"
```

## pipenv命令

```bash
$ pipenv
Usage: pipenv [OPTIONS] COMMAND [ARGS]...

Options:
  --update         Update Pipenv & pip to latest.     # 更新pipenv和pip到最新版本
  --where          Output project home information.   # 获取项目路径
  --venv           Output virtualenv information.     # 获取虚拟环境的路径
  --py             Output Python interpreter information. # 获取python解释器的路径
  --envs           Output Environment Variable options. # 输出当前的环境变量
  --rm             Remove the virtualenv.   # 删除虚拟环境
  --bare           Minimal output.
  --completion     Output completion (to be eval'd).
  --man            Display manpage.
  --three / --two  Use Python 3/2 when creating virtualenv. # 使用python3/2创建虚拟环境
  --python TEXT    Specify which version of Python virtualenv should use. # 指定python的版本信息
  --site-packages  Enable site-packages for the virtualenv.
  --jumbotron      An easter egg, effectively.
  --version        Show the version and exit.
  -h, --help       Show this message and exit.

Usage Examples:
   Create a new project using Python 3.6, specifically:
   $ pipenv --python 3.6

   Install all dependencies for a project (including dev):
   $ pipenv install --dev

   Create a lockfile containing pre-releases:
   $ pipenv lock --pre

   Show a graph of your installed dependencies:
   $ pipenv graph

   Check your installed dependencies for security vulnerabilities:
   $ pipenv check

   Install a local setup.py into your virtual environment/Pipfile:
   $ pipenv install -e .

Commands:
  check      Checks for security vulnerabilities and against PEP 508 markers
             provided in Pipfile.
  graph      Displays currently–installed dependency graph information.
  install    Installs provided packages and adds them to Pipfile, or (if none
             is given), installs all packages.
  lock       Generates Pipfile.lock.
  open       View a given module in your editor.
  run        Spawns a command installed into the virtualenv.
  shell      Spawns a shell within the virtualenv.
  uninstall  Un-installs a provided package and removes it from Pipfile.
  update     Uninstalls all packages, and re-installs package(s) in [packages]
             to latest compatible versions.

```

## 常用命令介绍

```bash
# 安装包
$ pipenv install

# 激活当前项目的虚拟环境
$ pipenv shell

# 安装开发依赖包
$ pipenv install pytest --dev

# 图形显示包依赖关系
$ pipenv graph

# 生成lockfile
$ pipenv lock

# 删除所有的安装包
$ pipenv uninstall --all
```

## 高级技巧

### 导入`requirements.txt`

当在执行`pipenv install`命令的时候，如果有一个`requirements.txt`文件，那么会自动从`requirements.txt`文件导入安装包信息并创建一个`Pipfile`文件。

同样可以使用`$ pipenv install -r path/to/requirements.txt`来导入`requirements.txt`文件

**注意**:

默认情况下，我们都是在`requirements.txt`文件里指定了安装包的版本信息的，在导入`requirements.txt`文件时，版本信息也会被自动写`Pipfile`文件里， 但是这个信息我们一般不需要保存在`Pipfile`文件里，需要手动更新`Pipfile`来删除版本信息


### 指定安装包的版本信息

为了安装指定版本的包信息，可以使用:

```bash
$ pipenv install requests==2.13.0
```

这个命令也会自动更新`Pipfile`文件

### 指定Python的版本信息

在创建虚拟环境的时候，我们可以指定使用的python版本信息，类似`pyenv`

```
$ pipenv --python 3
$ pipenv --python 3.6
$ pipenv --python 2.7.14
```

`pipenv`会自动扫描系统寻找合适的版本信息，如果找不到的话，同时又安装了`pyenv`, 它会自动调用`pyenv`下载对应的版本的python


### 指定安装包的源

如果我们需要在安装包时，从一个源下载一个安装包，然后从另一个源下载另一个安装包，我们可以通过下面的方式配置

```
[[source]]
url = "https://pypi.python.org/simple"
verify_ssl = true
name = "pypi"

[[source]]
url = "http://pypi.home.kennethreitz.org/simple"
verify_ssl = false
name = "home"

[dev-packages]

[packages]
requests = {version="*", index="home"}
maya = {version="*", index="pypi"}
records = "*"
```

如上设置了两个源：

* `pypi`(https://pypi.python.org/simple)
* `home`(http://pypi.home.kennethreitz.org/simple)

同时指定`requests`包从`home`源下载，`maya`包从`pypi`源下载

### 生成requirements.txt文件

我们也可以从`Pipfile`和`Pipfile.lock`文件来生成`requirements.txt`

```
# 生成requirements.txt文件
$ pipenv lock -r

# 生成dev-packages的requirements.txt文件
# pipenv lock -r -d
```

### 检查安全隐患
`pipenv`包含了`safety`模块，可以让我们坚持安装包是否存在安全隐患。

```bash
$ cat Pipfile
[packages]
django = "==1.10.1"

$ pipenv check
Checking PEP 508 requirements…
Passed!
Checking installed package safety…

33075: django >=1.10,<1.10.3 resolved (1.10.1 installed)!
Django before 1.8.x before 1.8.16, 1.9.x before 1.9.11, and 1.10.x before 1.10.3, when settings.DEBUG is True, allow remote attackers to conduct DNS rebinding attacks by leveraging failure to validate the HTTP Host header against settings.ALLOWED_HOSTS.

33076: django >=1.10,<1.10.3 resolved (1.10.1 installed)!
Django 1.8.x before 1.8.16, 1.9.x before 1.9.11, and 1.10.x before 1.10.3 use a hardcoded password for a temporary database user created when running tests with an Oracle database, which makes it easier for remote attackers to obtain access to the database server by leveraging failure to manually specify a password in the database settings TEST dictionary.

33300: django >=1.10,<1.10.7 resolved (1.10.1 installed)!
CVE-2017-7233: Open redirect and possible XSS attack via user-supplied numeric redirect URLs
============================================================================================

Django relies on user input in some cases  (e.g.
:func:`django.contrib.auth.views.login` and :doc:`i18n </topics/i18n/index>`)
to redirect the user to an "on success" URL. The security check for these
redirects (namely ``django.utils.http.is_safe_url()``) considered some numeric
URLs (e.g. ``http:999999999``) "safe" when they shouldn't be.

Also, if a developer relies on ``is_safe_url()`` to provide safe redirect
targets and puts such a URL into a link, they could suffer from an XSS attack.

CVE-2017-7234: Open redirect vulnerability in ``django.views.static.serve()``
=============================================================================

A maliciously crafted URL to a Django site using the
:func:`~django.views.static.serve` view could redirect to any other domain. The
view no longer does any redirects as they don't provide any known, useful
functionality.

Note, however, that this view has always carried a warning that it is not
hardened for production use and should be used only as a development aid.
```
### 编码风格检查

`pipenv`默认集成了`flake8`, 可以用来检测编码风格

```bash
$ cat t.py
import requests

$ pipenv check --style t.py
t.py:1:1: F401 'requests' imported but unused
t.py:1:16: W292 no newline at end of file
```

### 浏览模块代码

```bash
# 用编辑器打开requests模块
$ pipenv open requests
```

### 自动加载环境变量.env

如果项目根目录下有`.env`文件，`$ pipenv shell`和`$ pipenv run`会自动加载它。

```
$ cat .env
HELLO=WORLD

$ pipenv run python
Loading .env environment variables…
Python 2.7.13 (default, Jul 18 2017, 09:17:00)
[GCC 4.2.1 Compatible Apple LLVM 8.1.0 (clang-802.0.42)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import os
>>> os.environ['HELLO']
'WORLD'
```

### 自定义虚拟环境的路径

默认情况下,`pipenv`使用[`pew`](https://github.com/berdario/pew)来管理虚拟环境的路径，我们可以自定义`WORKON_HOME`环境变量来设置虚拟环境的路径。比如:

```bash
export WORKON_HOME=~/.venvs
```

我们也可以通过设置环境变量`PIPENV_VENV_IN_PROJECT`使虚拟环境在每个项目的根目录下`project/.venv`。


### 自动激活虚拟环境

配合[virtualenv-autodetect](https://github.com/RobertDeRose/virtualenv-autodetect)和设置`PIPENV_VENV_IN_PROJECT`环境变量可以自动激活虚拟环境。

在`.bashrc`或`.bash_profile`中配置如下

```
export PIPENV_VENV_IN_PROJECT=1
source /path/to/virtualenv-autodetect.sh
```

如果使用了`oh-my-zsh`, 可以直接使用它的插件形式

```
# 安装插件
$ git@github.com:RobertDeRose/virtualenv-autodetect.git ~/.oh-my-zsh/custom/plugins
```

再修改`.zshrc`文件启动插件

```
# 找到启动plugins的行添加启用插件
plugins=(... virtualenv-autodetect)
```

### 通过环境变量配置pipenv

pipenv内置了很多环境变量，可以通过设置这些环境变量来配置`pipenv`

```bash
$ pipenv --envs
The following environment variables can be set, to do various things:

  - PIPENV_MAX_DEPTH
  - PIPENV_SHELL
  - PIPENV_DOTENV_LOCATION
  - PIPENV_HIDE_EMOJIS
  - PIPENV_CACHE_DIR
  - PIPENV_VIRTUALENV
  - PIPENV_MAX_SUBPROCESS
  - PIPENV_COLORBLIND
  - PIPENV_VENV_IN_PROJECT # 在项目根路径下创建虚拟环境
  - PIPENV_MAX_ROUNDS
  - PIPENV_USE_SYSTEM
  - PIPENV_SHELL_COMPAT
  - PIPENV_USE_HASHES
  - PIPENV_NOSPIN
  - PIPENV_PIPFILE
  - PIPENV_INSTALL_TIMEOUT
  - PIPENV_YES
  - PIPENV_SHELL_FANCY
  - PIPENV_DONT_USE_PYENV
  - PIPENV_DONT_LOAD_ENV
  - PIPENV_DEFAULT_PYTHON_VERSION # 设置创建虚拟环境时默认的python版本信息，如: 3.6
  - PIPENV_SKIP_VALIDATION
  - PIPENV_TIMEOUT
```
需要修改某个默认配置时，只需要把它添加到`.bashrc`或`.bash_profile`文件里即可。


## 常见问题

### `pipenv install`时报错`pip.exceptions.InstallationError: Command "python setup.py egg_info" failed with error code 1`

错误原因是`pipenv`是用python2安装的，解决办法是使用pip3重新安装pipenv

```bash
$ pip unintall pipenv
$ pip3 install pipenv
```

### 在项目目录里运行`pipenv`时报错`AttributeError: module 'enum' has no attribute 'IntFlag'`

```bash
$ pipenv
Failed to import the site module
Traceback (most recent call last):
  File "/usr/local/Cellar/python3/3.6.4_2/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site.py", line 544, in <module>
    main()
  File "/usr/local/Cellar/python3/3.6.4_2/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site.py", line 530, in main
    known_paths = addusersitepackages(known_paths)
  File "/usr/local/Cellar/python3/3.6.4_2/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site.py", line 282, in addusersitepackages
    user_site = getusersitepackages()
  File "/usr/local/Cellar/python3/3.6.4_2/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site.py", line 258, in getusersitepackages
    user_base = getuserbase() # this will also set USER_BASE
  File "/usr/local/Cellar/python3/3.6.4_2/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site.py", line 248, in getuserbase
    USER_BASE = get_config_var('userbase')
  File "/usr/local/Cellar/python3/3.6.4_2/Frameworks/Python.framework/Versions/3.6/lib/python3.6/sysconfig.py", line 601, in get_config_var
    return get_config_vars().get(name)
  File "/usr/local/Cellar/python3/3.6.4_2/Frameworks/Python.framework/Versions/3.6/lib/python3.6/sysconfig.py", line 580, in get_config_vars
    import _osx_support
  File "/usr/local/Cellar/python3/3.6.4_2/Frameworks/Python.framework/Versions/3.6/lib/python3.6/_osx_support.py", line 4, in <module>
    import re
  File "/usr/local/Cellar/python3/3.6.4_2/Frameworks/Python.framework/Versions/3.6/lib/python3.6/re.py", line 142, in <module>
    class RegexFlag(enum.IntFlag):
AttributeError: module 'enum' has no attribute 'IntFlag'
```
是因为在项目目录里运行`pipenv`命令时，项目虚拟环境的python版本低于`3.6.4`, 由于`IntFlag`是从`python3.6.4`才开始集成到python内置模块的。当激活了项目的虚拟环境之后, 环境变量`PYTHONPATH`会被设置为当前虚拟环境的`site-packages`目录，因此`pipenv`依赖的`IntFlag`无法找到。 解决办法是在运行`pipenv`时设置环境变量`PYTHONPATH`为空

```bash
$ PYTHONPATH= pipenv
```
