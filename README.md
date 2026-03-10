# Turnover of products
## Problem:

- Retail inventory often suffers from slow-moving products and inefficient stock allocation, leading to capital being tied up in low-performing items.

## Solution:

- Cleaned and transformed retail data using Python (Pandas)

- Calculated inventory turnover using
- Identified slow-moving and overstocked products
- Performed ABC product classification based on sales contribution
- Analyzed product profitability and margin rates
- Built visualizations to analyze stock efficiency and product performance

## Impact
- Identified low-efficiency products with low turnover and low margins
- Highlighted high-performing products for prioritization
- Provided insights to optimize inventory management and product assortment

```python
import pandas as pd
sales = pd.read_excel("ДляОборачиваемости1.xlsx")
stock = pd.read_excel("ДляОборачиваемости2.xlsx")
sales = sales.iloc[6:]  
stock = stock.iloc[6:]
sales.columns = [
    "date", "store", "product_id", "product",
    "qty_sold", "sales_value", "margin"
]

stock.columns = [
    "product_id", "product", "store", "date",
    "avg_stock_qty", "avg_stock_value"
]

agg_sales = sales.groupby(['store','product']).agg(
    total_sold=('qty_sold','sum'),
    total_sales=('sales_value','sum'),
    total_margin=('margin','sum')
).reset_index()

agg_stock = stock.groupby(['store', 'product']).agg(
    avg_stock_qty=('avg_stock_qty','mean')
).reset_index()

df_turnover = pd.merge(
    agg_sales,
    agg_stock,
    left_on=['store','product'],
    right_on=['store','product'],
    how='left'
)

df_turnover['turnover'] = df_turnover['total_sold'] / df_turnover['avg_stock_qty']
df_turnover['avg_margin'] = df_turnover['total_margin'] / df_turnover['total_sales']
df_turnover = df_turnover.sort_values('turnover', ascending=False)
df_turnover.info()
cols = [
    "total_sold",
    "total_sales",
    "total_margin",
    "avg_stock_qty",
    "turnover",
    "avg_margin"
]

df_turnover[cols] = df_turnover[cols].apply(pd.to_numeric)
df_turnover.info()
slow_products = df_turnover[df_turnover["turnover"] < 1]
bad_products = df_turnover[
    (df_turnover["avg_stock_qty"] > df_turnover["avg_stock_qty"].median()) &
    (df_turnover["turnover"] < df_turnover["turnover"].median())
]
import matplotlib.pyplot as plt

plt.scatter(df_turnover["avg_stock_qty"], df_turnover["turnover"])

plt.xlabel("Average Stock")
plt.ylabel("Turnover")
plt.title("Stock vs Turnover")

plt.show()
df_turnover = df_turnover.sort_values("total_sales", ascending=False)

df_turnover["cum_sales"] = df_turnover["total_sales"].cumsum()

df_turnover["cum_perc"] = df_turnover["cum_sales"] / df_turnover["total_sales"].sum()

df_turnover["ABC"] = "C"

df_turnover.loc[df_turnover["cum_perc"] <= 0.8, "ABC"] = "A"
df_turnover.loc[
    (df_turnover["cum_perc"] > 0.8) &
    (df_turnover["cum_perc"] <= 0.95),
    "ABC"
] = "B"
df_turnover["margin_rate"] = df_turnover["total_margin"] / df_turnover["total_sales"]
top_margin_products = df_turnover.sort_values("margin_rate", ascending=False)
verybad_products = df_turnover[
    (df_turnover["turnover"] < df_turnover["turnover"].median()) &
    (df_turnover["margin_rate"] < df_turnover["margin_rate"].median())
]
plt.scatter(df_turnover["turnover"], df_turnover["margin_rate"])

plt.xlabel("Turnover")
plt.ylabel("Margin Rate")
plt.title("Product Efficiency")

plt.show()
best_products_by_store = df_turnover.loc[
    df_turnover.groupby("store")["total_sales"].idxmax()
]
best_products_by_store
```
