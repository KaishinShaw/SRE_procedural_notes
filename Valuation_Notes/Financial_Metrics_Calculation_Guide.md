------------------------------------------------------------------------

## 关键财务指标计算方法（Financial Analysis Dashboard）

以下列出dashboard中几个常用的核心财务指标及其计算方法：

> 部分使用了“安全除法”`safe_div`来避免除以 0 或 NaN

### 1. 增长率 (Growth Rate)

> **函数**：`growth_rate(current, previous)`

-   **公式**：\
    $$
    \text{Growth Rate} = \frac{\text{Current} - \text{Previous}}{\lvert \text{Previous} \rvert}
    $$
-   **意义**：用于衡量当前值与前期值之间的同比或环比变化幅度。\
-   **判断**：
    -   大于 0 表示正增长；\
    -   小于 0 表示负增长；\
    -   如果前期值为 0 或 NaN，则无法计算。

------------------------------------------------------------------------

### 2. 毛利率 (Gross Profit Margin)

> **变量**：`gross_profit_margin = gross_profit / revenue`

-   **公式**：\
    $$
    \text{GPM} = \frac{\text{Gross Profit}}{\text{Revenue}}
    $$
-   **意义**：毛利率越高，公司产品定价或成本控制能力越强。\
-   **判断**：
    -   一般行业的毛利率在 30% 到 40% 以上可以视为较好；R&D 公司可能波动较大。

------------------------------------------------------------------------

### 3. 营业利润率 (Operating Profit Margin)

> **变量**：`operating_profit_margin = operating_profit / revenue`

-   **公式**：\
    $$
    \text{OPM} = \frac{\text{Operating Profit}}{\text{Revenue}}
    $$
-   **意义**：衡量主营业务的盈利能力。\
-   **判断**：
    -   若长期为负，说明公司在主营层面可能长期亏损；需要关注经营效率及成本结构。

------------------------------------------------------------------------

### 4. 净利率 (Net Profit Margin)

> **变量**：`net_profit_margin = net_profit / revenue`

-   **公式**：\
    $$
    \text{NPM} = \frac{\text{Net Profit}}{\text{Revenue}}
    $$
-   **意义**：公司最终能将多少销售收入转化为净利润。\
-   **判断**：
    -   净利率过低或为负，需进一步审视费用结构与其它支出。

------------------------------------------------------------------------

### 5. EBITDA 及 EBITDA Margin

> **变量**：\
> - `ebitda = operating_profit + depreciation_amortization`\
> - `ebitda_margin = ebitda / revenue`

-   **公式**：\
    $$
    \text{EBITDA} = \text{Operating Profit} + \text{Depreciation and Amortization}
    $$\
    $$
    \text{EBITDA Margin} = \frac{\text{EBITDA}}{\text{Revenue}}
    $$
-   **意义**：EBITDA 能够在一定程度上排除固定资产折旧、摊销等非现金影响，更接近公司实际经营的现金流创造能力。\
-   **判断**：
    -   EBITDA Margin 越高，说明主营业务产生的现金流越充足。

------------------------------------------------------------------------

### 6. ROE (Return on Equity)

> **变量**：`roe = net_profit / total_equity`

-   **公式**：\
    $$
    \text{ROE} = \frac{\text{Net Profit}}{\text{Total Equity}}
    $$
-   **意义**：股东投入资本的回报率。\
-   **判断**：
    -   ROE 大于 10% 通常算是不错，但 R&D 型公司短期净亏损时，ROE 可能为负；如果公司“所有者权益”为负，ROE 也可能出现极端值或 NaN。

------------------------------------------------------------------------

### 7. ROA (Return on Assets)

> **变量**：`roa = net_profit / total_assets`

-   **公式**：\
    $$
    \text{ROA} = \frac{\text{Net Profit}}{\text{Total Assets}}
    $$
-   **意义**：衡量公司资产整体的获利能力。\
-   **判断**：
    -   ROA 为负或很低时，说明公司可能需要更有效地利用资产创造收益。

------------------------------------------------------------------------

### 8. ROIC (Return on Invested Capital)

> **变量**：`roic = operating_profit / (total_assets - current_liabilities)`

-   **公式**：\
    $$
    \text{ROIC} = \frac{\text{Operating Profit}}{\text{Invested Capital}}
    $$ 其中\
    $$
    \text{Invested Capital} = \text{Total Assets} - \text{Current Liabilities}
    $$
-   **意义**：衡量公司对投入资本的使用效率。\
-   **判断**：
    -   若公司持续 ROIC 低于加权平均资本成本(WACC)，意味着长期价值增值不足。

------------------------------------------------------------------------

### 9. Current Ratio (流动比率)

> **变量**：`current_ratio = current_assets / current_liabilities`

-   **公式**：\
    $$
    \text{Current Ratio} = \frac{\text{Current Assets}}{\text{Current Liabilities}}
    $$
-   **意义**：衡量公司使用流动资产偿还短期负债的能力。\
-   **判断**：
    -   通常认为 1.0 以下表示短期偿债风险较高；2.0 以上会相对安全。

------------------------------------------------------------------------

### 10. Quick Ratio (速动比率)

> **变量**：`quick_ratio = (current_assets - inventory) / current_liabilities`\
> （Dashboard 中没有提供库存信息，无区别于 Current Ratio）

-   **公式**：\
    $$
    \text{Quick Ratio} = \frac{\text{Current Assets} - \text{Inventory}}{\text{Current Liabilities}}
    $$
-   **意义**：衡量公司在不依赖存货变现的情况下能否偿还短期负债的能力。\
-   **判断**：
    -   一般 1.0 以上才较为稳健。

------------------------------------------------------------------------

### 11. Cash Ratio (现金比率)

> **变量**：`cash_ratio = cash_and_equivalents / current_liabilities`

-   **公式**：\
    $$
    \text{Cash Ratio} = \frac{\text{Cash & Equivalents}}{\text{Current Liabilities}}
    $$
-   **意义**：最保守的短期偿债能力指标，只考虑现金与等价物。\
-   **判断**：
    -   0.5 以下说明公司短期偿债能力相对薄弱；\
    -   1.0 以上说明公司以现金就能很轻松覆盖近期负债。

------------------------------------------------------------------------

### 12. Debt to Equity Ratio (资产负债率中的“负债/权益”)

> **变量**：`debt_to_equity = total_liabilities / total_equity`

-   **公式**：\
    $$
    \text{D/E} = \frac{\text{Total Liabilities}}{\text{Total Equity}}
    $$
-   **意义**：衡量公司财务杠杆水平。\
-   **判断**：
    -   高杠杆（2 以上）意味着更高的财务风险；\
    -   若公司股东权益为负，则比率可能为负或非常极端。

------------------------------------------------------------------------

### 13. Debt to Assets Ratio (负债占资产比率)

> **变量**：`debt_to_assets = total_liabilities / total_assets`

-   **公式**：\
    $$
    \text{Debt to Assets} = \frac{\text{Total Liabilities}}{\text{Total Assets}}
    $$
-   **意义**：公司总资产中有多大比重来自负债融资。\
-   **判断**：
    -   超过 70% 说明高度杠杆化；\
    -   太低也说明负债杠杆不足，需结合行业情况评估。

------------------------------------------------------------------------

### 14. Interest Coverage (利息保障倍数)

> **变量**：`interest_coverage = operating_profit / finance_costs(绝对值)`

-   **公式**：\
    $$
    \text{Interest Coverage} = \frac{\text{Operating Profit}}{\lvert \text{Finance Costs} \rvert}
    $$
-   **意义**：公司在盈利层面对利息支出的保障倍数。\
-   **判断**：
    -   小于 1 表示无法完全覆盖利息支出；\
    -   2～3 倍比较常见；高于 5 表示利息负担轻。

------------------------------------------------------------------------

### 15. Burn Rate & Cash Runway (现金消耗与可支撑月数)

> **Burn Rate**：每月运营现金流出量
>
> $$
> \text{Monthly Cash Burn} = \frac{|\text{Operating Cash Flow}|}{\text{Months}}
> $$ 若经营性现金流为负值才计算。
>
> **Cash Runway**：手头现金能支撑的月份数
>
> $$
> \text{Cash Runway (Months)} = \frac{\text{Cash and Equivalents}}{\text{Monthly Cash Burn}}
> $$

-   **意义**：对 R&D 型企业尤其重要，体现有多少个月的现金可以维持运营。\
-   **判断**：
    -   若 runway \< 12 个月，短期融资压力较大；\
    -   若 runway \> 24 个月，则相对较为安全。

------------------------------------------------------------------------

### 16. Intangible/Equity (无形资产占股东权益比)

> **变量**：`intangible_to_equity = intangible_assets / total_equity`

-   **公式**：\
    $$
    \text{Intangible to Equity} = \frac{\text{Intangible Assets}}{\text{Total Equity}}
    $$
-   **意义**：若无形资产占比过高，又无法快速变现，会加大资产价值的不确定性。

------------------------------------------------------------------------

### 17. Valuation Ratios (估值指标)

1.  **市盈率 (P/E)**\
    $$
    \text{P/E} = \frac{\text{Market Cap}}{\text{Profit to Parent}}
    $$

2.  **市净率 (P/B)**\
    $$
    \text{P/B} = \frac{\text{Market Cap}}{\text{Total Equity}}
    $$

3.  **市销率 (P/S)**\
    $$
    \text{P/S} = \frac{\text{Market Cap}}{\text{Revenue}}
    $$

4.  **EV/EBITDA**\
    $$
    \text{EV/EBITDA} = \frac{\text{Enterprise Value}}{\text{EBITDA}}
    $$

5.  **EV/Sales**\
    $$
    \text{EV/Sales} = \frac{\text{Enterprise Value}}{\text{Revenue}}
    $$

-   **判断**：
    -   对尚未盈利或周期性很强的公司，P/E 并不适用；\
    -   研发型公司常用 P/B 或 EV/EBITDA/Sales 等多角度比较；\
    -   优先结合行业平均值来判断高低。

------------------------------------------------------------------------

### 18. DCF (Discounted Cash Flow) 核心思路

``` none
企业价值 EV = 所有未来自由现金流的折现值 + 终值(若末年FCF>0)
股权价值 = 企业价值 + 现金 - 债务
每股价值 = 股权价值 / 总股本
```

-   **常见判断**：
    -   若 DCF 结果远高于当前市值，可能被低估，但需注意 R&D 型公司 FCF 为负的情况；\
    -   若 DCF 结果远低于当前股价，可能被高估，需要谨慎。

------------------------------------------------------------------------
