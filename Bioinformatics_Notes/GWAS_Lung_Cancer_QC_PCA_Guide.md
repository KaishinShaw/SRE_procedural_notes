# 使用PLINK2进行肺癌GWAS数据的质量控制及PCA分析

如何在Linux环境中使用PLINK2对肺癌GWAS数据进行质量控制(QC)并生成用于人群分层的PCA图

**假设条件：**

-   Linux环境PATH中已安装并配置好PLINK2
-   数据文件(`LC_final.bed`, `LC_final.bim`, `LC_final.fam`)已经是正确的PLINK二进制格式
-   熟悉基本的Linux命令行操作

**质量控制(QC)和PCA分析步骤：**

以下是结合最佳实践和常用PLINK2命令的结构化方法：

**1. 初始数据探索和汇总：**

在开始过滤前，了解数据集的基本特征非常重要。

``` bash
plink2 --bfile LC_final --freq --missing --het --out LC_initial_summary
```

-   `--bfile LC_final`：指定PLINK二进制文件的基本名称(不含.bed, .bim, .fam扩展名)
-   `--freq`：计算等位基因频率，用于后续过滤
-   `--missing`：计算每个个体和每个SNP的缺失率，用于识别基因分型质量差的样本或标记
-   `--het`：计算每个个体的杂合率，帮助识别杂合率异常高或低的个体，可能表示样本污染或近亲繁殖
-   `--out LC_initial_summary`：指定输出文件的前缀，PLINK2将创建文件如`LC_initial_summary.frq`、`LC_initial_summary.imiss`、`LC_initial_summary.lmiss`和`LC_initial_summary.het`

**检查输出文件：**

-   `.frq`：查看等位基因频率分布
-   `.imiss`和`.lmiss`：检查缺失率分布，任一文件中的高缺失率都是警示信号
-   `.het`：检查异常值

**2. 样本级别QC：**

此步骤移除不符合QC标准的个体(样本)。

-   **个体缺失率过滤(`--mind`)**：移除基因型缺失率高的个体。常用阈值为0.1(10%)，但您可能根据`.imiss`文件调整，0.05(5%)或0.02(2%)也是合理的选择。

-   **杂合率偏差过滤(`--het`过滤)**：移除杂合率极端的个体。通常计算平均杂合率，移除超出一定标准差范围的个体(如±3 SD)。第1步中的`--het`命令*计算*了杂合率，这里我们使用这些结果进行*过滤*。

    ``` bash
    # 基于缺失率过滤
    plink2 --bfile LC_final --mind 0.1 --make-bed --out LC_mind_filtered

    # 从.het文件计算杂合率的均值和标准差
    # (使用awk或类似工具；这不是plink2命令)
    awk '{sum+=$6; sumsq+=$6*$6} END {print "Mean:", sum/NR, "SD:", sqrt(sumsq/NR - (sum/NR)^2)}' LC_initial_summary.het > het_stats.txt

    # 使用het_stats.txt中的均值和标准差定义阈值
    # 例如，如果均值是0.2，标准差是0.05，那么：
    # 下限：0.2 - (3 * 0.05) = 0.05
    # 上限：0.2 + (3 * 0.05) = 0.35

    # 基于杂合率过滤(替换为您计算的边界值)
    plink2 --bfile LC_mind_filtered --het --extract-if-info Het < bounds.txt --make-bed --out LC_het_filtered
    ```

    `bounds.txt`文件应包含两行，分别为下限和上限。

-   **性别检查(`--check-sex`)**：识别报告性别与基因型性别不匹配的个体(基于X染色体杂合率)。这可以揭示样本交换或错误标记。

    ``` bash
    plink2 --bfile LC_het_filtered --check-sex --out LC_sexcheck
    ```

    检查`LC_sexcheck.sexcheck`文件。标记为`PROBLEM`的个体应该被调查并可能移除：

    ``` bash
    plink2 --bfile LC_het_filtered --remove LC_sexcheck.sexcheck --make-bed --out LC_sex_filtered
    ```

    **注意**：如果您的fam文件不包含性别信息，此步骤可能无法按预期工作。

**3. SNP级别QC：**

此步骤移除不符合QC标准的SNP(标记)。

-   **SNP缺失率过滤(`--geno`)**：移除基因型缺失率高的SNP。常用阈值为0.1(10%)，但与`--mind`类似，您可能使用0.05(5%)或0.02(2%)。通常对SNP级别缺失率比样本级别更严格。

-   **次等位基因频率(MAF)过滤(`--maf`)**：移除MAF非常低的SNP。罕见变异更容易出现基因分型错误且统计检验力有限。常用阈值为0.01(1%)或0.05(5%)。

-   **哈迪-温伯格平衡(HWE)过滤(`--hwe`)**：移除显著偏离HWE预期的SNP。偏离可能表示基因分型错误。通常*仅应用于对照组*，或在没有病例/对照信息时应用于整个样本。常用p值阈值为1e-6(0.000001)。

    ``` bash
    # 基于缺失率过滤
    plink2 --bfile LC_sex_filtered --geno 0.05 --make-bed --out LC_geno_filtered

    # 基于MAF过滤
    plink2 --bfile LC_geno_filtered --maf 0.01 --make-bed --out LC_maf_filtered

    # 基于HWE过滤(假设.fam文件中有病例/对照状态)
    # 提取对照组(表型 = 1)
    awk '$6 == 1 {print $1, $2}' LC_maf_filtered.fam > controls.txt
    plink2 --bfile LC_maf_filtered --filter-cases-controls controls.txt --hwe 1e-6 --make-bed --out LC_hwe_filtered

    # 如果没有病例/对照状态
    # plink2 --bfile LC_maf_filtered --hwe 1e-6 --make-bed --out LC_hwe_filtered
    ```

**4. 亲缘关系过滤(可选但推荐)：**

移除亲缘关系密切的个体，避免共同祖先导致的虚假关联。

-   **IBD(同一祖先)估计(`--genome`)**：PLINK2计算成对IBD系数(PI_HAT)。

-   **基于PI_HAT的过滤**：移除每对PI_HAT值高于阈值的个体中的一个(如0.2或0.125，代表二级亲属或更近)。

    ``` bash
    plink2 --bfile LC_hwe_filtered --genome --out LC_relatedness
    ```

    检查`LC_relatedness.genome`文件，识别PI_HAT值高的对。

    ``` bash
    # 示例：过滤PI_HAT > 0.2的个体
    plink2 --bfile LC_hwe_filtered --filter-pairwise-relatedness LC_relatedness.genome 0.2 --make-bed --out LC_relatedness_filtered
    ```

    此命令过滤PI_HAT值高于0.2的个体。

**5. 用于人群分层的主成分分析(PCA)：**

QC后，执行PCA以可视化人群结构。

-   **LD修剪(`--indep-pairwise`)**：PCA前，修剪连锁不平衡(LD)的SNP至关重要。LD可能扭曲PCA结果。此步骤选择很大程度上相互独立的SNP子集。

-   **PCA计算(`--pca`)**：计算主成分。

    ``` bash
    # LD修剪(常用参数，根据需要调整窗口大小和步长)
    plink2 --bfile LC_relatedness_filtered --indep-pairwise 50 5 0.2 --out LC_ld_pruned

    # 提取修剪后的SNP
    plink2 --bfile LC_relatedness_filtered --extract LC_ld_pruned.prune.in --make-bed --out LC_for_pca

    # 计算PCA
    plink2 --bfile LC_for_pca --pca --out LC_pca
    ```

**6. PCA可视化(使用R)：**

PLINK2生成`.eigenvec`(特征向量)和`.eigenval`(特征值)文件。使用R创建PCA图。

``` r
# 如果还没有安装必要的包，请先安装
# install.packages(c("ggplot2", "data.table"))

library(ggplot2)
library(data.table)

# 读取特征向量文件
eigenvec <- fread("LC_pca.eigenvec")

# 读取fam文件获取人群信息(如果有)
fam <- fread("LC_for_pca.fam")

# 将fam文件中的FID和IID添加到eigenvec数据
eigenvec$FID <- fam$V1
eigenvec$IID <- fam$V2

# 可选：添加人群标签(如果您有包含此信息的单独文件)
# population_info <- fread("population_labels.txt") # 示例文件
# eigenvec <- merge(eigenvec, population_info, by = c("FID", "IID"))

# 绘制PC1 vs PC2
ggplot(eigenvec, aes(x = PC1, y = PC2, color = FID)) +  # 假设FID包含人群信息
  geom_point() +
  xlab("PC1") +
  ylab("PC2") +
  ggtitle("肺癌GWAS数据的PCA图") +
  theme_bw()

# 碎石图，用于确定使用多少个PC
eigenval <- fread("LC_pca.eigenval")
eigenval_sum = sum(eigenval$`V1`)
prop_var = c()
for (i in (1:length(eigenval$V1))){
  prop_var[i] = eigenval[i,1]/eigenval_sum
}
plot(c(1:length(prop_var)), prop_var, xlab="PC", ylab="解释的方差比例", main="碎石图")
```

**关键考虑因素和解释：**

-   **操作顺序**：QC步骤的顺序很重要。例如，在检查HWE偏差之前，应先移除缺失率高的个体，因为缺失数据会影响HWE计算。
-   **阈值**：提供的阈值(如`--mind 0.1`，`--maf 0.01`)只是起点。您*必须*检查数据(汇总和QC步骤的输出文件)并适当调整这些阈值。对所有数据集都没有单一的"正确"阈值。
-   **LD修剪**：LD修剪对PCA至关重要，因为强LD中的SNP高度相关，这种相关性可能人为地夸大前几个主成分解释的方差。`--indep-pairwise`命令使用滑动窗口方法。`50 5 0.2`表示窗口大小为50个SNP，步长为5个SNP，r^2阈值为0.2。窗口内r^2 \> 0.2的任何SNP对将移除其中一个SNP。
-   **人群信息**：如果您有关于样本人群来源的信息(如种族)，在R图中包含此信息(作为颜色或形状美学)以帮助解释PCA结果。
-   **迭代QC**：QC通常是一个迭代过程。根据初步发现，您可能需要使用不同阈值重复某些步骤。
-   **病例/对照**：如果有病例/对照数据，确保仅使用对照组进行HWE过滤。
-   **.fam文件**：.fam文件应有6列：家庭ID、个体ID、父ID、母ID、性别(1=男，2=女，0=未知)、表型(1=对照，2=病例，0=缺失)。
