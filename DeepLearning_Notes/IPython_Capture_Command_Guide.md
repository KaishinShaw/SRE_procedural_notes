### 笔记：IPython的捕获魔术 %%capture 命令全攻略

本笔记将介绍 IPython 和 Jupyter Notebook 的一个强大功能 —— `%%capture` 魔术命令。这个命令允许我们捕获并保存单元格的输出，包括标准输出（stdout）和标准错误（stderr）。这对于创建干净的文档、分析输出、隐藏敏感信息或进行错误日志记录非常有用。

#### 一、环境简介

-   **IPython** 是一个增强的 Python 交互式解释器，支持丰富的交互式功能。
-   **Jupyter Notebook** 是一个开源的 Web 应用，它允许你创建和共享包含实时代码、方程、可视化和解释性文本的文档。

#### 二、%%capture 命令简介

`%%capture` 命令允许用户捕获单元格的输出，并可选择性地将输出保存到一个变量中。这一功能尤其适用于动态展示输出或对输出进行进一步处理的场景。

#### 三、基本语法

``` python
%%capture [variable_name]
```

-   **variable_name**（可选）：如果提供，捕获的输出将保存到这个变量中。如果省略，则输出将被丢弃。

#### 四、实验步骤

1.  **编写代码**：在 Jupyter Notebook 的单元格中输入 `%%capture`，后跟你希望执行并捕获输出的代码。
2.  **执行单元格**：执行包含 `%%capture` 的单元格，输出将被捕获并存储到指定的变量中（如果指定了变量）。

#### 五、实验示例

1.  **捕获标准输出**：

    ``` python
    %%capture
    print("Hello, World!")
    ```

    这个示例中，"Hello, World!" 将被捕获，不会显示在 Notebook 中。

2.  **捕获并保存输出**：

    ``` python
    %%capture output
    print("Hello, World!")
    print(output)
    ```

    在这个示例中，"Hello, World!" 将被捕获并保存到变量 `output` 中，然后可以再次打印出来。

3.  **结合图表库使用**：

    ``` python
    %%capture
    import matplotlib.pyplot as plt
    plt.plot([1, 2, 3], [4, 5, 6])
    plt.show()
    ```

    这将捕获图表的显示过程，但不会在 Notebook 中显示图表。

#### 六、注意事项

-   **资源消耗**：捕获大量输出可能会增加内存和处理时间的消耗。
-   **输出类型**：能捕获 stdout 和 stderr，但不包括图形界面的输出。
