# 财务分析指标计算方法（For Dashboard）

## 基础函数

### 安全除法 (safe_div)

``` python
def safe_div(a, b):
    """安全地计算 a / b，处理零和NaN的情况，适用于Series和标量值"""
```

安全除法函数用于避免除以零的错误，公式：

$$
\text{safe_div}(a, b) = 
\begin{cases} 
\frac{a}{b} & \text{if } b \neq 0 \text{ and } b \text{ is not NaN} \\
\text{NaN} & \text{otherwise}
\end{cases}
$$

### 增长率计算 (growth_rate)

``` python
def growth_rate(current, previous):
    """计算两个期间之间的增长率"""
```

增长率计算公式：

$$
\text{growth_rate}(current, previous) = 
\begin{cases} 
\frac{current - previous}{|previous|} & \text{if } previous \neq 0 \text{ and } previous \text{ is not NaN} \\
\text{NaN} & \text{otherwise}
\end{cases}
$$

## 1. 收益与增长指标

### 收入增长率

``` python
results['revenue_growth_yoy'] = growth_rate(current_rev, prev_rev)
```

$$
\text{收入增长率} = \frac{\text{当期收入} - \text{上期收入}}{|\text{上期收入}|}
$$

### EBITDA (息税折旧摊销前利润)

``` python
results['ebitda'] = results['operating_profit'] + results['depreciation_amortization'].fillna(0)
```

$$
\text{EBITDA} = \text{营业利润} + \text{折旧与摊销}
$$

### EBITDA增长率

``` python
results['ebitda_growth_yoy'] = growth_rate(current_ebitda, prev_ebitda)
```

$$
\text{EBITDA增长率} = \frac{\text{当期EBITDA} - \text{上期EBITDA}}{|\text{上期EBITDA}|}
$$

### 净利润增长率

``` python
results['net_income_growth_yoy'] = growth_rate(current_ni, prev_ni)
```

$$
\text{净利润增长率} = \frac{\text{当期净利润} - \text{上期净利润}}{|\text{上期净利润}|}
$$

### ROE (净资产收益率)

``` python
results['roe'] = safe_div(is_df['net_profit'], bs_df['total_equity'])
```

$$
\text{ROE} = \frac{\text{净利润}}{\text{所有者权益}}
$$

### ROA (资产收益率)

``` python
results['roa'] = safe_div(is_df['net_profit'], bs_df['total_assets'])
```

$$
\text{ROA} = \frac{\text{净利润}}{\text{总资产}}
$$

### ROIC (投资资本回报率)

``` python
invested_capital = bs_df['total_assets'] - bs_df['current_liabilities']
results['roic'] = safe_div(is_df['operating_profit'], invested_capital)
```

$$
\text{ROIC} = \frac{\text{营业利润}}{\text{投资资本}}
$$

其中，投资资本 = 总资产 - 流动负债

## 2. 盈利能力指标

### 毛利率

``` python
results['gross_profit_margin'] = safe_div(is_df['gross_profit'], is_df['revenue'])
```

$$
\text{毛利率} = \frac{\text{毛利润}}{\text{营业收入}}
$$

### 营业利润率

``` python
results['operating_profit_margin'] = safe_div(is_df['operating_profit'], is_df['revenue'])
```

$$
\text{营业利润率} = \frac{\text{营业利润}}{\text{营业收入}}
$$

### 净利润率

``` python
results['net_profit_margin'] = safe_div(is_df['net_profit'], is_df['revenue'])
```

$$
\text{净利润率} = \frac{\text{净利润}}{\text{营业收入}}
$$

### EBITDA利润率

``` python
results['ebitda_margin'] = safe_div(ebitda, is_df['revenue'])
```

$$
\text{EBITDA利润率} = \frac{\text{EBITDA}}{\text{营业收入}}
$$

### 资产周转率

``` python
results['asset_turnover'] = safe_div(is_df['revenue'], bs_df['total_assets'])
```

$$
\text{资产周转率} = \frac{\text{营业收入}}{\text{总资产}}
$$

### 研发强度代理指标

``` python
results['rd_intensity_proxy'] = safe_div(is_df['admin_expenses'], is_df['revenue'])
```

$$
\text{研发强度代理指标} = \frac{\text{管理费用}}{\text{营业收入}}
$$

## 3. 财务健康指标

### 流动比率

``` python
results['current_ratio'] = safe_div(bs_df['current_assets'], bs_df['current_liabilities'])
```

$$
\text{流动比率} = \frac{\text{流动资产}}{\text{流动负债}}
$$

**判断标准**:

-   \< 1: 短期偿债能力可能存在问题

-   1-2: 正常范围

-   \> 2: 较强的短期偿债能力

### 速动比率

``` python
results['quick_ratio'] = safe_div(bs_df['current_assets'], bs_df['current_liabilities'])
```

$$
\text{速动比率} = \frac{\text{流动资产}}{\text{流动负债}}
$$

**注**: 速动比率应扣除存货，但数据源中未包含存货数据，故dashboard中与流动比率计算相同

### 现金比率

``` python
results['cash_ratio'] = safe_div(bs_df['cash_and_equivalents'], bs_df['current_liabilities'])
```

$$
\text{现金比率} = \frac{\text{现金及现金等价物}}{\text{流动负债}}
$$

### 资产负债率

``` python
results['debt_to_assets'] = safe_div(bs_df['total_liabilities'], bs_df['total_assets'])
```

$$
\text{资产负债率} = \frac{\text{总负债}}{\text{总资产}}
$$

### 权益乘数

``` python
results['debt_to_equity'] = safe_div(bs_df['total_liabilities'], bs_df['total_equity'])
```

$$
\text{权益乘数} = \frac{\text{总负债}}{\text{所有者权益}}
$$

### 利息保障倍数

``` python
results['interest_coverage'] = safe_div(ebit, interest_expense)
```

$$
\text{利息保障倍数} = \frac{\text{息税前利润}}{\text{利息费用}}
$$

### 现金流对债务比率

``` python
results['cash_flow_to_debt'] = safe_div(cf_df['operating_cash_flow'], bs_df['total_liabilities'])
```

$$
\text{现金流对债务比率} = \frac{\text{经营活动现金流}}{\text{总负债}}
$$

### 月度现金消耗率

``` python
results['monthly_cash_burn'] = abs(op_cf) / months
```

$$
\text{月度现金消耗率} = \frac{|\text{经营活动现金流净额}|}{\text{月数}}
$$

### 现金可持续月数

``` python
results['cash_runway_months'] = cash / burn
```

$$
\text{现金可持续月数} = \frac{\text{现金及现金等价物}}{\text{月度现金消耗率}}
$$

**判断标准**:

-   \< 12月: 现金紧张，可能需要尽快融资

-   12-24月: 相对较短，应考虑融资选项

-   \> 24月: 现金充足，运营灵活性较高

## 4. 流动性指标

### 经营现金流比率

``` python
results['operating_cash_flow_ratio'] = safe_div(cf_df['operating_cash_flow'], bs_df['current_liabilities'])
```

$$
\text{经营现金流比率} = \frac{\text{经营活动现金流}}{\text{流动负债}}
$$

### 营运资金

``` python
results['working_capital'] = bs_df['current_assets'] - bs_df['current_liabilities']
```

$$
\text{营运资金} = \text{流动资产} - \text{流动负债}
$$

### 营运资金占总资产比例

``` python
results['working_capital_to_assets'] = safe_div(bs_df['current_assets'] - bs_df['current_liabilities'], bs_df['total_assets'])
```

$$
\text{营运资金占总资产比例} = \frac{\text{流动资产} - \text{流动负债}}{\text{总资产}}
$$

### 现金占总资产比例

``` python
results['cash_to_total_assets'] = safe_div(bs_df['cash_and_equivalents'], bs_df['total_assets'])
```

$$
\text{现金占总资产比例} = \frac{\text{现金及现金等价物}}{\text{总资产}}
$$

### 短期投资占总资产比例

``` python
results['short_term_investments_to_assets'] = safe_div(short_term_inv, bs_df['total_assets'])
```

$$
\text{短期投资占总资产比例} = \frac{\text{短期投资}}{\text{总资产}}
$$

## 5. 偿债能力指标

### 长期负债占总资产比例

``` python
results['long_term_debt_to_assets'] = safe_div(bs_df['non_current_liabilities'], bs_df['total_assets'])
```

$$
\text{长期负债占总资产比例} = \frac{\text{非流动负债}}{\text{总资产}}
$$

### 权益乘数

``` python
results['equity_multiplier'] = safe_div(bs_df['total_assets'], bs_df['total_equity'])
```

$$
\text{权益乘数} = \frac{\text{总资产}}{\text{所有者权益}}
$$

### 债务保障倍数

``` python
total_debt_service = bs_df['short_term_loans'] + bs_df['long_term_loans'].fillna(0)
results['debt_service_coverage'] = safe_div(is_df['operating_profit'], total_debt_service)
```

$$
\text{债务保障倍数} = \frac{\text{营业利润}}{\text{短期借款} + \text{长期借款}}
$$

## 6. 无形资产指标

### 无形资产占总资产比例

``` python
results['intangible_to_total_assets'] = safe_div(bs_df['intangible_assets'], bs_df['total_assets'])
```

$$
\text{无形资产占总资产比例} = \frac{\text{无形资产}}{\text{总资产}}
$$

### 每股无形资产

``` python
results['intangible_per_share'] = safe_div(bs_df['intangible_assets'], bs_df['share_capital'])
```

$$
\text{每股无形资产} = \frac{\text{无形资产}}{\text{股本}}
$$

### 无形资产占权益比例

``` python
results['intangible_to_equity'] = safe_div(bs_df['intangible_assets'], bs_df['total_equity'])
```

$$
\text{无形资产占权益比例} = \frac{\text{无形资产}}{\text{所有者权益}}
$$

### 无形资产同比增长率

``` python
results['intangible_growth_yoy'] = growth_rate(current, previous)
```

$$
\text{无形资产同比增长率} = \frac{\text{当期无形资产} - \text{上期无形资产}}{|\text{上期无形资产}|}
$$

### 研发收入比例代理指标

``` python
results['rd_to_revenue_proxy'] = safe_div(is_df['admin_expenses'], is_df['revenue'])
```

$$
\text{研发收入比例代理指标} = \frac{\text{管理费用}}{\text{营业收入}}
$$

### 无形资产占非流动资产比例

``` python
results['intangible_to_non_current_assets'] = safe_div(bs_df['intangible_assets'], bs_df['non_current_assets'])
```

$$
\text{无形资产占非流动资产比例} = \frac{\text{无形资产}}{\text{非流动资产}}
$$

## 7. 估值指标

### 市盈率 (P/E)

``` python
results['pe_ratio'] = safe_div(market_cap, is_df['profit_to_parent'])
```

$$
\text{市盈率} = \frac{\text{市值}}{\text{归属于母公司股东的净利润}}
$$

### 市净率 (P/B)

``` python
results['pb_ratio'] = safe_div(market_cap, bs_df['total_equity'])
```

$$
\text{市净率} = \frac{\text{市值}}{\text{所有者权益}}
$$

### 市销率 (P/S)

``` python
results['ps_ratio'] = safe_div(market_cap, is_df['revenue'])
```

$$
\text{市销率} = \frac{\text{市值}}{\text{营业收入}}
$$

### 每股净资产

``` python
results['book_value_per_share'] = safe_div(bs_df['total_equity'], bs_df['share_capital'])
```

$$
\text{每股净资产} = \frac{\text{所有者权益}}{\text{股本}}
$$

### 每股收益 (EPS)

``` python
results['eps'] = safe_div(is_df['profit_to_parent'], bs_df['share_capital'])
```

$$
\text{每股收益} = \frac{\text{归属于母公司股东的净利润}}{\text{股本}}
$$

### 企业价值 (EV)

``` python
# 对于有市值数据的期间
results['enterprise_value'] = market_cap + bs_df['total_liabilities'] - bs_df['cash_and_equivalents']

# 对于无市值数据的期间
results['enterprise_value'] = bs_df['total_liabilities'] - bs_df['cash_and_equivalents']
```

$$
\text{企业价值} = 
\begin{cases} 
\text{市值} + \text{总负债} - \text{现金及现金等价物} & \text{若有市值数据} \\
\text{总负债} - \text{现金及现金等价物} & \text{若无市值数据}
\end{cases}
$$

### EV/EBITDA

``` python
results['ev_to_ebitda'] = safe_div(results['enterprise_value'], ebitda)
```

$$
\text{EV/EBITDA} = \frac{\text{企业价值}}{\text{EBITDA}}
$$

### EV/销售额

``` python
results['ev_to_sales'] = safe_div(results['enterprise_value'], is_df['revenue'])
```

$$
\text{EV/销售额} = \frac{\text{企业价值}}{\text{营业收入}}
$$

### 每股营收

``` python
results['revenue_per_share'] = safe_div(is_df['revenue'], bs_df['share_capital'])
```

$$
\text{每股营收} = \frac{\text{营业收入}}{\text{股本}}
$$

### 每股自由现金流

``` python
fcf = cf_df['operating_cash_flow'] - cf_df['capital_expenditure']
results['fcf_per_share'] = safe_div(fcf, bs_df['share_capital'])
```

$$
\text{每股自由现金流} = \frac{\text{经营活动现金流} - \text{资本支出}}{\text{股本}}
$$

## 8. 现金流折现模型 (DCF)

### 自由现金流 (FCF)

``` python
latest_fcf = latest_cf['operating_cash_flow'] - latest_cf['capital_expenditure']
```

$$
\text{自由现金流} = \text{经营活动现金流} - \text{资本支出}
$$

### 终值计算

``` python
terminal_value = forecasted_fcf[-1] * (1 + terminal_growth_rate) / (discount_rate - terminal_growth_rate)
```

$$
\text{终值} = \frac{\text{最终预测年度FCF} \times (1 + \text{永续增长率})}{\text{折现率} - \text{永续增长率}}
$$

### 预测期现金流现值

``` python
present_value = fcf / ((1 + discount_rate) ** (i + 1))
```

$$
\text{预测期现金流现值}_i = \frac{\text{预测期FCF}_i}{(1 + \text{折现率})^{i+1}}
$$

### 终值现值

``` python
present_value_terminal = terminal_value / ((1 + discount_rate) ** forecast_years)
```

$$
\text{终值现值} = \frac{\text{终值}}{(1 + \text{折现率})^{\text{预测年数}}}
$$

### 企业价值 (DCF)

``` python
enterprise_value = sum(present_value_fcf) + present_value_terminal
```

$$
\text{企业价值} = \sum \text{预测期现金流现值} + \text{终值现值}
$$

### 股权价值

``` python
equity_value = enterprise_value + cash - debt
```

$$
\text{股权价值} = \text{企业价值} + \text{现金及现金等价物} - \text{总债务}
$$

### 每股价值

``` python
per_share_value = equity_value / shares_outstanding
```

$$
\text{每股价值} = \frac{\text{股权价值}}{\text{流通股数}}
$$

### DCF与市值比较

``` python
dcf_market_ratio = equity_value / market_cap
```

$$
\text{DCF与市值比率} = \frac{\text{DCF估值}}{\text{当前市值}}
$$

**判断标准**:

-   \> 1.2: DCF估值高于市值20%以上，判断为股票被低估

-   0.8 - 1.2: DCF估值与市值接近，判断为合理

-   \< 0.8: DCF估值低于市值20%以上，判断为股票被高估

### 现金可持续年数

``` python
cash_runway = cash / burn_rate
```

$$
\text{现金可持续年数} = \frac{\text{现金及现金等价物}}{|\text{经营活动现金流净额}|}
$$

**判断标准**:

-   \< 1年: 判断为极度紧张，融资压力较大

-   1-2年: 判断为相对紧张，应考虑融资选项

-   \> 2年: 判断为现金充足，运营灵活性较高
