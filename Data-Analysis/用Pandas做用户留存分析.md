---
title: 用 Pandas 做用户留存分析
---

```python
%matplotlib inline
import matplotlib 
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
sns.set(rc={"figure.figsize": (16, 10)})
matplotlib.rc('font', family='simhei')
```

我们聚合出了一份 2019 年度的用户下单样例数据，有三个字段，首单月，下单月，用户量


```python
df1 = pd.read_csv("./data/order_user_cnt_cohort_sample_data.csv")
df1.head()
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>首单年月</th>
      <th>下单年月</th>
      <th>user_cnt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2019-01</td>
      <td>2019-01</td>
      <td>88760</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-01</td>
      <td>2019-02</td>
      <td>9552</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019-02</td>
      <td>2019-02</td>
      <td>87155</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2019-03</td>
      <td>2019-03</td>
      <td>101254</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2019-02</td>
      <td>2019-03</td>
      <td>10453</td>
    </tr>
  </tbody>
</table>
</div>



创建透视表，其实透视表就可以看出留存情况了，只是没那么好理解


```python
df2 = df1.pivot_table(
    index=['首单年月'], columns='下单年月', values='user_cnt')
df2
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>下单年月</th>
      <th>2019-01</th>
      <th>2019-02</th>
      <th>2019-03</th>
      <th>2019-04</th>
      <th>2019-05</th>
      <th>2019-06</th>
      <th>2019-07</th>
      <th>2019-08</th>
      <th>2019-09</th>
      <th>2019-10</th>
      <th>2019-11</th>
      <th>2019-12</th>
    </tr>
    <tr>
      <th>首单年月</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2019-01</th>
      <td>88760.0</td>
      <td>9552.0</td>
      <td>8207.0</td>
      <td>6403.0</td>
      <td>5178.0</td>
      <td>4894.0</td>
      <td>3343.0</td>
      <td>3063.0</td>
      <td>2443.0</td>
      <td>1966.0</td>
      <td>3212.0</td>
      <td>2278.0</td>
    </tr>
    <tr>
      <th>2019-02</th>
      <td>NaN</td>
      <td>87155.0</td>
      <td>10453.0</td>
      <td>6388.0</td>
      <td>5511.0</td>
      <td>4673.0</td>
      <td>3335.0</td>
      <td>3024.0</td>
      <td>2503.0</td>
      <td>1815.0</td>
      <td>2889.0</td>
      <td>1991.0</td>
    </tr>
    <tr>
      <th>2019-03</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>101254.0</td>
      <td>10673.0</td>
      <td>8068.0</td>
      <td>6835.0</td>
      <td>4769.0</td>
      <td>4180.0</td>
      <td>3531.0</td>
      <td>2671.0</td>
      <td>4078.0</td>
      <td>2841.0</td>
    </tr>
    <tr>
      <th>2019-04</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>102449.0</td>
      <td>10497.0</td>
      <td>9495.0</td>
      <td>5869.0</td>
      <td>5164.0</td>
      <td>4244.0</td>
      <td>3177.0</td>
      <td>5519.0</td>
      <td>3600.0</td>
    </tr>
    <tr>
      <th>2019-05</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>89913.0</td>
      <td>11471.0</td>
      <td>6562.0</td>
      <td>5420.0</td>
      <td>4240.0</td>
      <td>3090.0</td>
      <td>5278.0</td>
      <td>3611.0</td>
    </tr>
    <tr>
      <th>2019-06</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>82524.0</td>
      <td>7230.0</td>
      <td>5491.0</td>
      <td>4311.0</td>
      <td>3013.0</td>
      <td>6027.0</td>
      <td>3579.0</td>
    </tr>
    <tr>
      <th>2019-07</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>66243.0</td>
      <td>6927.0</td>
      <td>4457.0</td>
      <td>3309.0</td>
      <td>5105.0</td>
      <td>3090.0</td>
    </tr>
    <tr>
      <th>2019-08</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>63119.0</td>
      <td>5325.0</td>
      <td>3454.0</td>
      <td>4656.0</td>
      <td>3245.0</td>
    </tr>
    <tr>
      <th>2019-09</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>55891.0</td>
      <td>4336.0</td>
      <td>5745.0</td>
      <td>3748.0</td>
    </tr>
    <tr>
      <th>2019-10</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>45593.0</td>
      <td>5389.0</td>
      <td>3550.0</td>
    </tr>
    <tr>
      <th>2019-11</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>69600.0</td>
      <td>5860.0</td>
    </tr>
    <tr>
      <th>2019-12</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>47870.0</td>
    </tr>
  </tbody>
</table>
</div>



现在我们按下单月顺序排列


```python
def cohort_period(df):
    df['CohortPeriod'] = np.arange(len(df)) + 1
    return df

cohorts = df1.groupby('首单年月').apply(cohort_period)
cohorts.head()
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>首单年月</th>
      <th>下单年月</th>
      <th>user_cnt</th>
      <th>CohortPeriod</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2019-01</td>
      <td>2019-01</td>
      <td>88760</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-01</td>
      <td>2019-02</td>
      <td>9552</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019-02</td>
      <td>2019-02</td>
      <td>87155</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2019-03</td>
      <td>2019-03</td>
      <td>101254</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2019-02</td>
      <td>2019-03</td>
      <td>10453</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>


```python
cohorts.set_index(['首单年月', 'CohortPeriod'], inplace=True)
cohorts.head()
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>下单年月</th>
      <th>user_cnt</th>
    </tr>
    <tr>
      <th>首单年月</th>
      <th>CohortPeriod</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="2" valign="top">2019-01</th>
      <th>1</th>
      <td>2019-01</td>
      <td>88760</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019-02</td>
      <td>9552</td>
    </tr>
    <tr>
      <th>2019-02</th>
      <th>1</th>
      <td>2019-02</td>
      <td>87155</td>
    </tr>
    <tr>
      <th>2019-03</th>
      <th>1</th>
      <td>2019-03</td>
      <td>101254</td>
    </tr>
    <tr>
      <th>2019-02</th>
      <th>2</th>
      <td>2019-03</td>
      <td>10453</td>
    </tr>
  </tbody>
</table>
</div>




```python
cohort_group_size = cohorts['user_cnt'].groupby('首单年月').first()
cohort_group_size
```




    首单年月
    2019-01     88760
    2019-02     87155
    2019-03    101254
    2019-04    102449
    2019-05     89913
    2019-06     82524
    2019-07     66243
    2019-08     63119
    2019-09     55891
    2019-10     45593
    2019-11     69600
    2019-12     47870
    Name: user_cnt, dtype: int64




```python
cohorts['user_cnt'].unstack(0)
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>首单年月</th>
      <th>2019-01</th>
      <th>2019-02</th>
      <th>2019-03</th>
      <th>2019-04</th>
      <th>2019-05</th>
      <th>2019-06</th>
      <th>2019-07</th>
      <th>2019-08</th>
      <th>2019-09</th>
      <th>2019-10</th>
      <th>2019-11</th>
      <th>2019-12</th>
    </tr>
    <tr>
      <th>CohortPeriod</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>88760.0</td>
      <td>87155.0</td>
      <td>101254.0</td>
      <td>102449.0</td>
      <td>89913.0</td>
      <td>82524.0</td>
      <td>66243.0</td>
      <td>63119.0</td>
      <td>55891.0</td>
      <td>45593.0</td>
      <td>69600.0</td>
      <td>47870.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>9552.0</td>
      <td>10453.0</td>
      <td>10673.0</td>
      <td>10497.0</td>
      <td>11471.0</td>
      <td>7230.0</td>
      <td>6927.0</td>
      <td>5325.0</td>
      <td>4336.0</td>
      <td>5389.0</td>
      <td>5860.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8207.0</td>
      <td>6388.0</td>
      <td>8068.0</td>
      <td>9495.0</td>
      <td>6562.0</td>
      <td>5491.0</td>
      <td>4457.0</td>
      <td>3454.0</td>
      <td>5745.0</td>
      <td>3550.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6403.0</td>
      <td>5511.0</td>
      <td>6835.0</td>
      <td>5869.0</td>
      <td>5420.0</td>
      <td>4311.0</td>
      <td>3309.0</td>
      <td>4656.0</td>
      <td>3748.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5178.0</td>
      <td>4673.0</td>
      <td>4769.0</td>
      <td>5164.0</td>
      <td>4240.0</td>
      <td>3013.0</td>
      <td>5105.0</td>
      <td>3245.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>4894.0</td>
      <td>3335.0</td>
      <td>4180.0</td>
      <td>4244.0</td>
      <td>3090.0</td>
      <td>6027.0</td>
      <td>3090.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>3343.0</td>
      <td>3024.0</td>
      <td>3531.0</td>
      <td>3177.0</td>
      <td>5278.0</td>
      <td>3579.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>3063.0</td>
      <td>2503.0</td>
      <td>2671.0</td>
      <td>5519.0</td>
      <td>3611.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2443.0</td>
      <td>1815.0</td>
      <td>4078.0</td>
      <td>3600.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1966.0</td>
      <td>2889.0</td>
      <td>2841.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11</th>
      <td>3212.0</td>
      <td>1991.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2278.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>

转换成百分比


```python
user_retention = cohorts['user_cnt'].unstack(0).divide(cohort_group_size, axis=1)
user_retention
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>首单年月</th>
      <th>2019-01</th>
      <th>2019-02</th>
      <th>2019-03</th>
      <th>2019-04</th>
      <th>2019-05</th>
      <th>2019-06</th>
      <th>2019-07</th>
      <th>2019-08</th>
      <th>2019-09</th>
      <th>2019-10</th>
      <th>2019-11</th>
      <th>2019-12</th>
    </tr>
    <tr>
      <th>CohortPeriod</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.107616</td>
      <td>0.119936</td>
      <td>0.105408</td>
      <td>0.102461</td>
      <td>0.127579</td>
      <td>0.087611</td>
      <td>0.104570</td>
      <td>0.084364</td>
      <td>0.077580</td>
      <td>0.118198</td>
      <td>0.084195</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.092463</td>
      <td>0.073295</td>
      <td>0.079681</td>
      <td>0.092680</td>
      <td>0.072982</td>
      <td>0.066538</td>
      <td>0.067283</td>
      <td>0.054722</td>
      <td>0.102789</td>
      <td>0.077863</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.072138</td>
      <td>0.063232</td>
      <td>0.067504</td>
      <td>0.057287</td>
      <td>0.060280</td>
      <td>0.052239</td>
      <td>0.049952</td>
      <td>0.073765</td>
      <td>0.067059</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.058337</td>
      <td>0.053617</td>
      <td>0.047099</td>
      <td>0.050406</td>
      <td>0.047157</td>
      <td>0.036511</td>
      <td>0.077065</td>
      <td>0.051411</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.055137</td>
      <td>0.038265</td>
      <td>0.041282</td>
      <td>0.041425</td>
      <td>0.034367</td>
      <td>0.073033</td>
      <td>0.046646</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.037663</td>
      <td>0.034697</td>
      <td>0.034873</td>
      <td>0.031011</td>
      <td>0.058701</td>
      <td>0.043369</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.034509</td>
      <td>0.028719</td>
      <td>0.026379</td>
      <td>0.053871</td>
      <td>0.040161</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.027524</td>
      <td>0.020825</td>
      <td>0.040275</td>
      <td>0.035139</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>0.022150</td>
      <td>0.033148</td>
      <td>0.028058</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11</th>
      <td>0.036187</td>
      <td>0.022844</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.025665</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>

通过颜色渐变的方式用热图展示出来，现在我们可以对比第三个月的留存情况

```python
plt.figure(figsize=(20, 10))
plt.title('用户下单留存分析')
sns.heatmap(user_retention.T, mask=user_retention.T.isnull(), annot=True, cmap="rocket_r", fmt='.2%');
```

![file](https://img.toutiao.io/attachment/cda6df9b949d4e2ca816e0a1cc3ae3b4/w600)

> HowTos 项目 Github 地址： https://github.com/toutiaoio/HowTos