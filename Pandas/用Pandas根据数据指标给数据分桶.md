---
title: 用 Pandas 根据数据指标给数据分桶
url: https://toutiao.io/posts/9f3gljk
---

在数据分析需求中，通常需要看数据的分布，比如我们想根据用户维表上的成长值来给用户划分级别


```python
import pandas as pd
```

从数仓中读取数据


```python
df = pd.read_sql("select user_id, growth_point from dim_user", conn)
```


```python
df.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_id</th>
      <th>growth_point</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2685</td>
      <td>2548</td>
    </tr>
    <tr>
      <th>1</th>
      <td>167142</td>
      <td>198</td>
    </tr>
    <tr>
      <th>2</th>
      <td>49927</td>
      <td>1256</td>
    </tr>
    <tr>
      <th>3</th>
      <td>264364</td>
      <td>1494</td>
    </tr>
    <tr>
      <th>4</th>
      <td>344132</td>
      <td>187</td>
    </tr>
  </tbody>
</table>

定义用户级别及范围


```python
bins = [0, 500, 4000, 10000, 50000, 150000, 800000, 1500000]
labels = ['V0', 'V1', 'V2', 'V3', 'V4', 'V5', 'V6']
```

用 Pandas 的 `cut` 方法给 df 增加 `level` 字段表示用户级别， `right=False` 表示左闭右开区间


```python
df['level'] = pd.cut(df['growth_point'], bins, labels=labels, right=False)
```


```python
df.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_id</th>
      <th>growth_point</th>
      <th>level</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2685</td>
      <td>2548</td>
      <td>V1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>167142</td>
      <td>198</td>
      <td>V0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>49927</td>
      <td>1256</td>
      <td>V1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>264364</td>
      <td>1494</td>
      <td>V1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>344132</td>
      <td>187</td>
      <td>V0</td>
    </tr>
  </tbody>
</table>


统计每个用户级别的用户数分布


```python
df.groupby('level')['user_id'].count()
```




    level
    V0    1660916
    V1     329702
    V2      30733
    V3      11326
    V4        452
    V5         71
    V6         20
    Name: user_id, dtype: int64

