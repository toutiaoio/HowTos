---
title: 用 Pandas 做库龄分析
---

### 库龄分析

库存周转率是在某特定的周期，销售成本与存货平均余额的比率。用以衡量一定时期内存货资产的周转速度，是反映企业的供应链的整体效率的绩效指标之一，而且很多企业都把它作为整体经营业绩的考核指标之一。

#### 1. 库存账龄分析的概念

提到库存账龄，就不得不说到库存周转率。库存周转率是在某特定的周期，销售成本与存货平均余额的比率。用以衡量一定时期内存货资产的周转速度，是反映企业的供应链的整体效率的绩效指标之一，而且很多企业都把它作为整体经营业绩的考核指标之一。而库存账龄是在某时间节点，某种或者某类存货的库存时间的加权平均值。很明显，库存周转率越高，库存账龄越低，但是二者又不是反比关系（比较简单的证明就是同样的平均库存，入库时间的不同就会引起库存账龄很大的差异），所以虽然这二者有着千丝万缕的联系，但是不能简单的把库存账龄看成库存周转率的一个衍生指标来对待。

#### 2. 库存账龄分析的目的

在库存账龄分析中,其目的主要有以下两点：

一、库存成本的控制。与应收账款的账龄一样，存货的库存账龄越长，说明周转越慢，占压的资金也就越多。这也就是我们大家平常所说的滞销品。对于滞销品，应该分析其原因，从计划的源头控制入手，才能最有效的降低无效的库存，达到降低库存总额的目的。滞销品实际上包括两部分：不周转的物料和周转慢的物料，对于不周转的物料，显然除了上述工作外，还应该做相应的处理：比如代用或者变卖。

二、对于超龄的库存，也应该做好存货损失的准备，更真实的反映库存的实际价值。

#### 3. 库存账龄的计算

手工计算库存账龄是很困难的。在ERP系统中，库存账龄的计算相当方便。如果要计算某一仓库或者全部存货的库存账龄（虽然该数字可能没有实际的意义），那么公式如下：

    库存账龄=∑(批次入库数量*批次入库时间/统计时点库存总额)。

下面我举例一说明：

在2008年10月6日，存货A的库存为1000，系统自动搜索入库单，得到如下入库单：

    入库单号 日期 数量
    NO.025 10月5日入库200；
    NO.023 10月4日入库300；
    NO.022 10月3日入库300；
    NO.020 10月2日入库400；
    NO.015 10月1日入库100。
    
系统默认先进先出的原则，从10月6日的入库单倒推满1000为止，也就是200（10月5日）+300（10月4日）+300（10月3日）+200（10月2日入库的400只取值200即可）=1000.
那么：`库存账龄 = 200/1000*1 + 300/1000*2 + 300/1000*3 + 200/1000*4 = 0.2 + 0.6 + 0.9 + 0.8 = 2.5天`

库存账龄还有另外一种报表方式，反映的是库存账龄的集中度，举例二如下：

库存30天以下的数量： 1000；
库存30天--90天的数量： 2000；
库存90天--180天的数量：1500；
库存180天以上的数量： 500。

https://baike.baidu.com/item/%E5%BA%93%E9%BE%84%E5%88%86%E6%9E%90


```python
import pandas as pd
from datetime import datetime
```

我们清洗好了一份采购明细样例数据, 按采购日期降序排列


```python
df = pd.read_csv('./data/stockage-sample-data.csv')
df.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sku_id</th>
      <th>当前库存</th>
      <th>采购入库日期</th>
      <th>采购入库数量</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1007271270102</td>
      <td>1296</td>
      <td>2021-02-22</td>
      <td>1296</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1008903694810</td>
      <td>1388</td>
      <td>2021-02-22</td>
      <td>1388</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1006671011405</td>
      <td>1296</td>
      <td>2021-02-22</td>
      <td>1296</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1008229644607</td>
      <td>240</td>
      <td>2021-02-22</td>
      <td>240</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1006379474960</td>
      <td>180</td>
      <td>2021-02-22</td>
      <td>180</td>
    </tr>
  </tbody>
</table>

我们先计算出采购日期距离今天有多少天

```python
current_datetime = datetime.now()
print(current_datetime)
```

    2021-03-02 14:12:40.513775


```python
df['days'] = (current_datetime - pd.to_datetime(df['采购入库日期'])).dt.days
df.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sku_id</th>
      <th>当前库存</th>
      <th>采购入库日期</th>
      <th>采购入库数量</th>
      <th>days</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1007271270102</td>
      <td>1296</td>
      <td>2021-02-22</td>
      <td>1296</td>
      <td>8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1008903694810</td>
      <td>1388</td>
      <td>2021-02-22</td>
      <td>1388</td>
      <td>8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1006671011405</td>
      <td>1296</td>
      <td>2021-02-22</td>
      <td>1296</td>
      <td>8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1008229644607</td>
      <td>240</td>
      <td>2021-02-22</td>
      <td>240</td>
      <td>8</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1006379474960</td>
      <td>180</td>
      <td>2021-02-22</td>
      <td>180</td>
      <td>8</td>
    </tr>
  </tbody>
</table>


现在我们需要对每一个 SKU 计算他的库龄， 我们可以通过group-apply的方式


```python
def calc_stockage(gdf):
    
    if gdf.iloc[0]['当前库存']==0:
        return 0
                                           
    gdf['c0'] = gdf['采购入库数量'].cumsum()  # 累计采购入库数量                                       
    gdf['c1'] = gdf['c0'].shift(1).fillna(0) # 累计到前一次的采购入库数量
    
    def calc_belongs_current_row(row):
        if (row['当前库存'] - row['c0']) > 0:
            return row['采购入库数量']
        else:
            bc = row['当前库存'] - row['c1']
            return bc if bc>0 else 0
   
    gdf['c2'] = gdf.apply(calc_belongs_current_row, axis=1) # 当前库存中属于这次采购的部分 
    return gdf.apply(lambda x: int(x['c2']/x['当前库存'] * x['days']), axis=1).sum()

df_stockage = df.groupby('sku_id').apply(calc_stockage).to_frame(name='stockage').reset_index()
df_stockage.head()

```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sku_id</th>
      <th>stockage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1000014008898</td>
      <td>85</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1000015356938</td>
      <td>540</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1000015695798</td>
      <td>610</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1000020598014</td>
      <td>453</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1000020677953</td>
      <td>22</td>
    </tr>
  </tbody>
</table>


```python
df[df.sku_id==1000014008898]
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sku_id</th>
      <th>当前库存</th>
      <th>采购入库日期</th>
      <th>采购入库数量</th>
      <th>days</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>49</th>
      <td>1000014008898</td>
      <td>170</td>
      <td>2021-02-08</td>
      <td>15</td>
      <td>22</td>
    </tr>
    <tr>
      <th>208</th>
      <td>1000014008898</td>
      <td>170</td>
      <td>2021-01-22</td>
      <td>55</td>
      <td>39</td>
    </tr>
    <tr>
      <th>912</th>
      <td>1000014008898</td>
      <td>170</td>
      <td>2020-10-29</td>
      <td>2000</td>
      <td>124</td>
    </tr>
    <tr>
      <th>1153</th>
      <td>1000014008898</td>
      <td>170</td>
      <td>2020-09-18</td>
      <td>300</td>
      <td>165</td>
    </tr>
    <tr>
      <th>1355</th>
      <td>1000014008898</td>
      <td>170</td>
      <td>2020-08-21</td>
      <td>380</td>
      <td>193</td>
    </tr>
    <tr>
      <th>1439</th>
      <td>1000014008898</td>
      <td>170</td>
      <td>2020-07-31</td>
      <td>200</td>
      <td>214</td>
    </tr>
    <tr>
      <th>1717</th>
      <td>1000014008898</td>
      <td>170</td>
      <td>2020-05-21</td>
      <td>150</td>
      <td>285</td>
    </tr>
    <tr>
      <th>1737</th>
      <td>1000014008898</td>
      <td>170</td>
      <td>2020-05-18</td>
      <td>450</td>
      <td>288</td>
    </tr>
  </tbody>
</table>


```python
df_stockage.shape
```

    (1090, 2)


每个SKU的库龄已经算出来了，现在我们来看一下他们的分布情况


```python
df_stockage['stockage_group'] = pd.cut(df_stockage['stockage'], (0, 30, 60, 90, 180, 365, 365*5), right=False)
df_stockage.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sku_id</th>
      <th>stockage</th>
      <th>stockage_group</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1000014008898</td>
      <td>85</td>
      <td>[60, 90)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1000015356938</td>
      <td>540</td>
      <td>[365, 1825)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1000015695798</td>
      <td>610</td>
      <td>[365, 1825)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1000020598014</td>
      <td>453</td>
      <td>[365, 1825)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1000020677953</td>
      <td>22</td>
      <td>[0, 30)</td>
    </tr>
  </tbody>
</table>

每个 SKU 的当前库存数


```python
df_stock = df.groupby('sku_id')[['当前库存']].first().reset_index()
df_stock.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sku_id</th>
      <th>当前库存</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1000014008898</td>
      <td>170</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1000015356938</td>
      <td>409</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1000015695798</td>
      <td>111</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1000020598014</td>
      <td>407</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1000020677953</td>
      <td>472</td>
    </tr>
  </tbody>
</table>


我们来看一下库龄分布情况


```python
df_stockage.merge(df_stock).groupby('stockage_group').agg({
    'sku_id': pd.Series.nunique,
    'stockage': ['std', 'mean', 'median', 'max', 'min'],
    '当前库存': 'sum'
})
```

<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th>sku_id</th>
      <th colspan="5" halign="left">stockage</th>
      <th>当前库存</th>
    </tr>
    <tr>
      <th></th>
      <th>nunique</th>
      <th>std</th>
      <th>mean</th>
      <th>median</th>
      <th>max</th>
      <th>min</th>
      <th>sum</th>
    </tr>
    <tr>
      <th>stockage_group</th>
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
      <th>[0, 30)</th>
      <td>46</td>
      <td>6.119937</td>
      <td>20.543478</td>
      <td>21</td>
      <td>29</td>
      <td>7</td>
      <td>22746</td>
    </tr>
    <tr>
      <th>[30, 60)</th>
      <td>158</td>
      <td>6.660796</td>
      <td>39.506329</td>
      <td>38</td>
      <td>59</td>
      <td>30</td>
      <td>58949</td>
    </tr>
    <tr>
      <th>[60, 90)</th>
      <td>74</td>
      <td>9.038002</td>
      <td>75.283784</td>
      <td>74</td>
      <td>89</td>
      <td>60</td>
      <td>28867</td>
    </tr>
    <tr>
      <th>[90, 180)</th>
      <td>215</td>
      <td>24.437695</td>
      <td>125.004651</td>
      <td>123</td>
      <td>178</td>
      <td>90</td>
      <td>68854</td>
    </tr>
    <tr>
      <th>[180, 365)</th>
      <td>226</td>
      <td>40.574263</td>
      <td>304.836283</td>
      <td>309</td>
      <td>364</td>
      <td>180</td>
      <td>99461</td>
    </tr>
    <tr>
      <th>[365, 1825)</th>
      <td>371</td>
      <td>199.385834</td>
      <td>595.126685</td>
      <td>504</td>
      <td>1160</td>
      <td>367</td>
      <td>131362</td>
    </tr>
  </tbody>
</table>

> HowTos 项目 Github 地址： https://github.com/toutiaoio/HowTos