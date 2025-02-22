## SAS Unique identification record Methods

------------------------------------------------------------------------

### A. `PROC SORT` 语句（使用 `NODUPKEY` 选项和 `OUT=` 参数）

``` sas
proc sort data=input_dataset nodupkey out=output_dataset;
    by key_variable;
run;
```

1.  **`proc sort data=input_dataset`**
    -   启动排序过程，对数据集 `input_dataset` 进行排序。
2.  **`nodupkey`**
    -   此选项会根据接下来的 `by` 语句中所指定的变量来检查数据集中的“重复”观测值。如果多条记录在 `by` 变量上完全相同，SAS 只保留第一条记录，舍弃其余重复观测。
3.  **`out=output_dataset`**
    -   将排序后的结果保存到新的数据集 `output_dataset` 里。这样既可以保留原始数据集，又得到去重后的排序数据集。
4.  **`by key_variable;`**
    -   指定用于排序和判断重复的关键变量（也可以有多个变量）。SAS 根据这些变量进行排序，并在 `nodupkey` 的作用下保留每个组合的第一条记录。
5.  **`run;`**
    -   执行上述排序过程。

------------------------------------------------------------------------

### B. `DATA` 步骤中使用 `IF FIRST.Customer_Id=1`

``` sas
data new_dataset;
    set dataset;
    by Customer_Id;
    if first.Customer_Id = 1;
run;
```

1.  **`data new_dataset;`**
    -   开始一个数据步，指定新的输出数据集名称为 `new_dataset`。
2.  **`set dataset;`**
    -   从已有的数据集 `dataset` 中读取数据。
3.  **`by Customer_Id;`**
    -   指定数据是按照 `Customer_Id` 进行分组的（前提是数据集必须先按 `Customer_Id` 排序，否则此语句不起作用）。SAS 在此基础上自动创建两个临时变量：
    -   **`first.Customer_Id`**：表示当前组中第一条记录，当遇到每个新组时其值为 1（真）。
    -   **`last.Customer_Id`**：表示当前组中最后一条记录。
4.  **`if first.Customer_Id = 1;`**
    -   使用该判断条件，只保留每个 `Customer_Id` 组中的第一条记录。这样可以达到去重或筛选分组首个观测值的效果。
5.  **`run;`**
    -   执行数据步，生成新的数据集 `new_dataset`。

------------------------------------------------------------------------

### C. `PROC SQL` 中使用 `SELECT DISTINCT` 语句

``` sas
proc sql;
    create table new_table as 
    select distinct *
    from dataset;
quit;
```

1.  **`proc sql;`**
    -   进入 PROC SQL 模式，允许使用 SQL 语言来操作 SAS 数据集。
2.  **`create table new_table as`**
    -   指定创建一个新的数据表 `new_table`。
3.  **`select distinct *`**
    -   `distinct` 关键字用于消除查询结果中的重复记录。\
    -   当使用 `*` 时，SAS 会将数据集所有列的数据作为一个整体来判断重复，只有完全不重复的行才会保留。
4.  **`from dataset;`**
    -   指定查询的数据来源是数据集 `dataset`。
5.  **`quit;`**
    -   退出 PROC SQL 模式，结束 SQL 过程。

------------------------------------------------------------------------

### 总结说明

-   **使用 `PROC SORT` + `NODUPKEY`**：适合当你明确知道哪些变量可以唯一标识记录、并且想借助排序过程去掉重复记录时使用。\
-   **使用 `DATA` 步骤中的 `IF FIRST.Customer_Id=1`**：通常在数据已按 `Customer_Id` 排序的前提下，通过自动创建的临时变量筛选出每个组的第一条记录。\
-   **使用 `PROC SQL` 的 `SELECT DISTINCT`**：适合直接通过 SQL 语句语义获取去重的数据集，其操作更接近标准 SQL 语法，且代码较为简洁。
