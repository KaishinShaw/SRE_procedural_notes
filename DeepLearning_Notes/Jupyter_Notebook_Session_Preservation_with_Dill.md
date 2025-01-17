# 如何在重启电脑后仍保持 Jupyter Notebook 的进度

当你重启电脑时，可能会担心丢失当前的 Jupyter Notebook 会话进度。幸运的是，Dill 包可以帮助你保存并在需要时恢复会话状态。

是否曾因重启电脑而忘记保存你重要的 Jupyter Notebook？“这本笔记本运行了很长时间，现在却全部丢失了。”

我深有体会！这种情况我也遇到过好几次。

通常，我只能建议“重启内核并重新运行所有单元格”。不过，有了这个工具，下次就不需要这样做了。

## 认识 Dill

Dill 提供了一个简单的方式来保存和恢复你的 Jupyter Notebook 会话。

安装 Dill 非常简单，只需如下操作：

``` bash
pip install dill
```

保存一个 Jupyter Notebook 会话，执行以下命令：

``` python
import dill
dill.dump_session('notebook_env.db')
```

要恢复之前的会话，使用：

``` python
import dill
dill.load_session('notebook_env.db')
```

Dill 同样适用于普通的 Python 解释器会话。

## 局限性

使用 Dill 时，需要注意以下几点局限性：

-   不能用于保存生成器。
-   保存的会话不包含图形输出，需要重新执行代码来生成。
-   有用户反馈，在不同的机器上恢复会话可能会失败。

## 关于 Dill

Dill 是对 Python 的 pickle 模块的扩展，支持序列化和反序列化大多数内置的 Python 类型。序列化是将对象转换成字节流的过程，反之亦然。

Dill 提供了与 pickle 相同的接口，并增加了一些额外的功能。除了可以对 Python 对象进行序列化，Dill 还能够保存整个解释器的会话状态。这意味着你可以保存一个会话状态，关闭解释器，将序列化文件传输到另一台计算机，再打开新的解释器，加载之前的会话状态，从而继续之前的工作。

Dill 的主要应用是将 Python 对象以字节流的形式跨网络传输，而非仅仅是本地文件存储。Dill 非常灵活，支持序列化用户自定义的类和函数。不过，Dill 并不保证对恶意数据的安全性，使用者需要确保解序列化的数据来自可信源。

## 主要特性

Dill 支持序列化以下类型：

-   基本数据类型如 None, Type, Bool, Int, Long, Float, Complex, Str, Unicode
-   复合数据类型如 Tuple, List, Dict, File, Buffer, Builtin
-   老式和新式的类及其实例
-   Set, Frozenset, Array, Functions, Exceptions

Dill 还能处理一些更复杂的类型：

-   带有 yield 的函数，嵌套函数，lambda 表达式
-   Cell, Method, Unbound Method, Module, Code, Method Wrapper
-   Method Descriptor, GetSet Descriptor, Member Descriptor, Wrapper Descriptor, Xrange, Slice, NotImplemented, Ellipsis, Quit

目前，Dill 不能处理以下类型：

-   Frame, Generator, Traceback

Dill 还提供了其他功能：

-   保存和加载 Python 解释器会话
-   提取函数和类的源代码
-   交互式地诊断序列化错误

## 我的观点

Dill 是一个非常有用的工具，尤其是在需要重启电脑或预防 Jupyter Kernel 可能崩溃的情况下。虽然不推荐在每个 Jupyter Notebook 中都使用 Dill，但它在需要时可以非常有用。同时，也要注意，由于 Dill 无法处理生成器，使用时需谨慎。

更多信息和功能介绍，请访问 [Dill 官方文档](https://github.com/uqfoundation/dill)。
