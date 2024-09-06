---
layout: article
title: 解析 Python 是如何找包的
date: 2024-09-06
key: P2024-09-06
tags: Python
show_author_profile: true
comment: true
sharing: true
aside:
  toc: true
---

解析 Python 包的寻找与安装。

<!--more-->

# 前言

有几个常见的问题：

- 为什么我用 Pycharm 能运行，而在 cmd 里却运行不了？
- import module 为什么报 `ModuleNotFound`？
- 安装了 pip 为什么运行报找不到可执行文件？

要解决这类问题，就需要得知道 Python 是如何找包的。

# Python 是如何寻找包的

现在大家的电脑上很可能不只有一个 Python，还有更多的虚拟环境。一不小心你就可能在安装包的时候忘记注意安装包的路径了。

假如你的 Python 解释器的路径是 `$path_prefix/bin/python`，那么你启动 Python 交互环境，或者用这个解释器运行脚本时，会默认寻找以下位置：

1. `$path_prefix/lib`（标准库路径）
2. `$path_prefix/lib/pythonX.Y/site-packages`（三方库路径，X.Y 是对应 Python 的主次版本号，如 3.7, 2.6）
3. 当前工作目录（`pwd`命令的返回结果）

对于`$path_prefix`的值取决于 Python 是如何安装的：如果使用的是 Linux 上自带预装的 Python 则其值通常为 `/usr`；如果你自己从源码编译 Python 并使用默认的安装选项则其值通常是 `/usr/local`。

以上路径为 Unix 习惯，如果是 Windows 系统则标准库路径 `$path_prefix/bin` 为 `$path_prefix/Scripts`

## 相关属性

- `sys.executable`：当前使用的 Python 解释器路径
- `sys.prefix`：当前使用的 `$path_prefix`
- `sys.path`：当前包的搜索路径列表

除此之外，还可以在*命令行*中运行 `python -m site`，会打印出当前 Python 的一些信息，包括搜索路径列表。

```
>>> import sys
>>> sys.executable
'/usr/bin/python3'
>>> sys.prefix
'/usr'
>>> sys.path
['', '/usr/lib/python310.zip', '/usr/lib/python3.10', '/usr/lib/python3.10/lib-dynload', '/usr/local/lib/python3.10/dist-packages', '/usr/lib/python3/dist-packages']
```

## 如何安装包

安装 Python 包基本是用的 `pip`。就算是用 `pipenv`，`poetry`，底层依然是 `pip`。

运行 `pip` 有两种方式：

1. `pip ...`

2. `python -m pip ...`

这两种安装方式大同小异，通常推荐使用第二种方式，特别是在有多个 Python 版本时。

当直接调用 `pip` 命令时，`pip` 会根据自身文件中的shebang（脚本文件最顶部`#!`开头的一行字符串），确定用于执行 `pip` 的 Python 解释器是哪个。通常`pip` 的 shebang 将指向同一目录下的 Python 解释器，例如 pip 的安装路径是 `$path_prefix/bin/pip`，那么这个 pip 的 shebang 指向的即是同一目录下的 Python 解释器路径`$path_prefix/bin/python`。

想要确认可以通过运行 `cat $(which pip)`，即可查看当前使用的`pip` 文件。其第一行即为使用的 Python 解释器路径，例如`#!/usr/bin/python3`。

但是，当系统不同路径中存在多个 Python 版本时，直接使用`pip`命令可能会导致混淆。即 pip 的 shebang 指向的不是你想要使用的 Python 解释器，导致安装到了非预期的地方。

使用`python -m pip`的好处即是，会明确地使用当前环境指定的 Python 解释器来运行 `pip`。这里 `-m` 参数是让 Python 运行库模块 `pip`。

无论使用哪种方式运行 `pip` ，包都会自动安装到 `$path_prefix/lib/pythonX.Y/site-packages` 下，可执行程序安装到 `$path_prefix/bin` 下。

## 虚拟环境

虚拟环境就是为了隔离不同项目的依赖包，使他们安装到不同的路径下，以防止依赖冲突的问题。

虚拟环境通过创建一个独立的目录结构，包含一个项目所需的全部 Python 执行文件和所有依赖库的副本。使得每个项目都可以有自己的依赖版本，不会与系统全局安装的包或其他项目的依赖冲突。

例如，当运行`python -m venv .venv `创建新的环境`.venv`时，它会创建一个独立的目录结构，包括`.venv/bin`，`.venv/lib/pythonX.Y/site-packages`等目录。

执行`source .ven/bin/activate`后，将会把 `.ven/bin`的路径加在环境变量`PATH`的最前面，使其优先于系统路径。之后当调用 Python 解释器或 pip 时，使用的将是虚拟环境里的版本，从而实现了安装路径的环境隔离。

# 脚本运行方式对搜索路径的影响

从上面Python是如何寻找包的介绍大家可以知道，当 Python 找不找得到一个包时，最直接的原因是 `sys.path` 的路径。

当我们运行代码时，脚本不同的运行方式会影响到 `sys.path` 从而造成不同的行为，下面我们就来讨论这个问题。

举例来解释更容易，假设你的包结构如下：

```
.
├── main.py
└── my_package
    ├── __init__.py
    ├── a.py
    └── b.py
```

`main.py` 的内容很简单：

```python
import sys
print(sys.path)

import my_package.a
```

`a.py` 为：

```python
import sys
print("I'm a")
print(sys.path)
```

在 `main.py` 同级的目录(根目录)下执行

```
$ python main.py
['/home/ittuann/test_path', ...]  # 省略的路径是共同的，与讨论的问题无关
I'm a
['/home/ittuann/test_path', ...]
$
$ python my_package/a.py
I'm a
['/home/ittuann/test_path/my_package', ...]
```

`python xxx.py` 的运行方式叫做**直接运行**。IDE 中的「 Run File」、「 运行脚本」用的就是这种方式。

可以看到直接运行时 `sys.path`的第一个值是**该脚本文件所在的目录**，随脚本路径而变化。

然后，我们写了新的文件`b.py`继续开发这个项目，并在 `a.py` 中导入新的 `b.py`：

`b.py` 为：

```python
print("I'm b")
```

`a.py`修改为（写成这格式是为了方便演示搜索路径）：

```python
import sys
print("I'm a")
print(sys.path)

import b
```

那么再执行一遍上面的测试：

```
$ python main.py
['/home/ittuann/test_path', ...]
Traceback (most recent call last):
  File "/home/ittuann/test_path/main.py", line 4, in <module>
    import my_package.a
  File "/home/ittuann/test_path/my_package/a.py", line 5, in <module>
    import b
ModuleNotFoundError: No module named 'b'
$
$ python my_package/a.py
I'm a
['/home/ittuann/test_path/my_package', ...]
I'm b
```

第一个测试出错了，这个报错就是意料之中了——`sys.path` 压根没有 `b.py` 所在的目录 `/home/ittuann/test_path/my_package`，当然找不到 `b` 了。

那么改为使用**相对导入**，即`from my_package import b`，或`from . import b`：

`a.py`更新为

```python
import sys
print("I'm a")
print(sys.path)

from my_package import b
```

再执行一遍上面的测试：

```
$ python main.py
['/home/ittuann/test_path', ...]
I'm a
['/home/ittuann/test_path', ...]
I'm b
$
$ python my_package/a.py
I'm a
['/home/ittuann/test_path/my_package', ...]
Traceback (most recent call last):
  File "C:\Downloads\tests\my_package\a.py", line 5, in <module>
    from my_package import b
ModuleNotFoundError: No module named 'my_package'
```

使用`from . import b`导入，执行结果也是和上面一样，即第一个运行成功第二个运行失败：

```
$ python my_package/a.py
I'm a
['/home/ittuann/test_path/my_package', ...]
Traceback (most recent call last):
  File "C:\Downloads\tests\my_package\a.py", line 5, in <module>
    from . import b
ImportError: attempted relative import with no known parent package
```

## 正确的推荐做法

当我们需要运行子目录中某脚本的代码时，这是让这两次运行都成功的方法。：

- 应该用 `python -m <module_name>`运行，这种使用 `-m` 标志运行方式叫做**以模块方式运行**脚本。

- 同时 `a.py` 中导入 `b` 的语句应为 `from my_package import b`。

即在本例中 `a.py`的正确写法是：

```python
import sys
print("I'm a")
print(sys.path)

from my_package import b
```

然后使用`python -m my_package.a`执行`a.py`：

```
$ python main.py  # 和python -m main效果一样
['/home/ittuann/test_path', ...]
I'm a
['/home/ittuann/test_path', ...]
I'm b
$
$ python -m my_package.a
I'm a
['/home/ittuann/test_path', ...]
I'm b
```

`python -m main` 和 `python main.py` 的效果是一样的。

现在可以看到这两次运行的 `sys.path `内容一致了，它的第一个值是**当前运行所在的目录**。

`python -m` 后面的参数是（以 `.` 分隔的）**模块名**，而不是路径名。

以模块方式运行脚本，可以帮助实现导入路径的一致性，从而避免相对导入和路径问题。无论脚本被放置在项目的哪个位置，导入时都会基于项目的根目录进行，而不是基于脚本所在的目录。

这也是为什么 Django 官方文档中推荐导入名称全部用 `myapp.models.users` 这种形式，导入都是从项目的根目录开始的。这种导入方式不依赖于文件在项目中的位置。好处是无论你在项目的哪个文件中，都可以使用相同的导入语句来引用同一个模块，特别是在复杂和层次分明的项目中能减少由于文件移动或重构导致的导入错误。

参考链接:

> https://frostming.com/2019/03-13/where-do-your-packages-go/
