# Turnover of products
## What have i done:

-Trimmed and removed the defects of the tables

-Gave the columns names

-Grouped by store and product 

-Merged two tables 

-Estimated turnover
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
```
