---
title: 用 Pandas 计算日志中事件的时间间隔
url: https://toutiao.io/posts/o73tkeh
---

![file](https://img.toutiao.io/attachment/18c599fbfd924453b429eaabacf6dc21/w600)

通常我们需要对日志数据清洗出一些事件间隔的特征，给数据分析或者机器学习建模提供数据支持。

假设我们有一些事件日志，这些日志记录了用户在App中事件类型、时间戳、用户ID等其他一些数据，像下面这样:


```python
data = """user_id,event_time,event_type
100003,2021-01-18 01:06:00,onLaunch
100001,2021-01-19 08:30:00,onLaunch
100003,2021-01-20 02:39:00,onLaunch
100003,2021-01-20 08:15:00,onLaunch
100002,2021-01-20 10:05:00,onLaunch
100001,2021-01-20 14:40:00,onLaunch
100001,2021-01-20 18:05:00,onLaunch
100002,2021-01-21 21:11:00,onLaunch
100003,2021-01-22 10:05:00,onLaunch
100001,2021-01-23 09:18:00,onLaunch
100003,2021-01-23 17:35:00,onLaunch
100001,2021-01-25 16:49:00,onLaunch
100003,2021-01-26 12:13:00,onLaunch
100001,2021-01-27 19:56:00,onLaunch"""
```

我们把日志读取到 Pandas 的内存表 DataFrame 中．真实环境可以通过 `read_sql` 或者 `read_csv` 等方法读取数据源


```python
import pandas as pd
ldata = [x.split(',') for x in data.split('\n')]
df = pd.DataFrame(ldata[1:], columns=ldata[0])
df['event_time'] =  pd.to_datetime(df['event_time'])
df.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_id</th>
      <th>event_time</th>
      <th>event_type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>100003</td>
      <td>2021-01-18 01:06:00</td>
      <td>onLaunch</td>
    </tr>
    <tr>
      <th>1</th>
      <td>100001</td>
      <td>2021-01-19 08:30:00</td>
      <td>onLaunch</td>
    </tr>
    <tr>
      <th>2</th>
      <td>100003</td>
      <td>2021-01-20 02:39:00</td>
      <td>onLaunch</td>
    </tr>
    <tr>
      <th>3</th>
      <td>100003</td>
      <td>2021-01-20 08:15:00</td>
      <td>onLaunch</td>
    </tr>
    <tr>
      <th>4</th>
      <td>100002</td>
      <td>2021-01-20 10:05:00</td>
      <td>onLaunch</td>
    </tr>
  </tbody>
</table>


我们希望提取出每个用户的启动次数（`lifetime_launchs`）、昨日启动次数(`yesterday_launchs`) 以及距离上次登录的天数 (`days_since_last_launch`)。

使用 pandas，我们按照用户和日期进行汇总，得到每个用户每天的启动次数。


```python
user_launchs = (df.set_index('event_time')
               .groupby(['user_id', pd.Grouper(freq='D')])
               .size()
               .rename('launchs'))
user_launchs
```




    user_id  event_time
    100001   2021-01-19    1
             2021-01-20    2
             2021-01-23    1
             2021-01-25    1
             2021-01-27    1
    100002   2021-01-20    1
             2021-01-21    1
    100003   2021-01-18    1
             2021-01-20    2
             2021-01-22    1
             2021-01-23    1
             2021-01-26    1
    Name: launchs, dtype: int64



可以看出，这里只有有启动的日志，日期不连续，为了提取出上面的特征，我们需要填充没有启动的日期.
把用户放到一个连续的时间轴上.


```python
# 创建一个日期范围作为 DataFrame 的索引
dates = pd.date_range(df.event_time.min().date(),
                      df.event_time.max().date(),
                      freq='1D')

# 取出去重的用户ID
users = df['user_id'].unique()


# 通过用户和日期交叉创建出一个 MultiIndex
idx = pd.MultiIndex.from_product([users, dates], names=['user_id', 'event_time'])

# 索引建索引
user_launchs = user_launchs.reindex(idx)

user_launchs.head()
```




    user_id  event_time
    100003   2021-01-18    1.0
             2021-01-19    NaN
             2021-01-20    2.0
             2021-01-21    NaN
             2021-01-22    1.0
    Name: launchs, dtype: float64



这样我们构建了一个连续的时间序列．现在可以提取我们需要的特征了.


```python
# 为空的数据行把空填充0,并转成 DataFrame
user_launchs = user_launchs.fillna(0).to_frame()
user_launchs.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>launchs</th>
    </tr>
    <tr>
      <th>user_id</th>
      <th>event_time</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">100003</th>
      <th>2021-01-18</th>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2021-01-19</th>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2021-01-20</th>
      <td>2.0</td>
    </tr>
    <tr>
      <th>2021-01-21</th>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2021-01-22</th>
      <td>1.0</td>
    </tr>
  </tbody>
</table>


通过`shift`方法向下移动一行作为下一行的前一天，给 user_launchs 增加 `launchs_yesterday` 字段


```python
user_launchs['launchs_yesterday'] = user_launchs.groupby(level='user_id')['launchs'].shift(1)
user_launchs.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>launchs</th>
      <th>launchs_yesterday</th>
    </tr>
    <tr>
      <th>user_id</th>
      <th>event_time</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">100003</th>
      <th>2021-01-18</th>
      <td>1.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2021-01-19</th>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2021-01-20</th>
      <td>2.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2021-01-21</th>
      <td>0.0</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>2021-01-22</th>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>

滚动的累计出当前日期下每个用户的总启动次数，给 user_launchs 增加 `lifetime_launchs` 字段．


```python
user_launchs['lifetime_launchs'] = (user_launchs
                                  .groupby(level='user_id')
                                  .launchs.cumsum()
                                  .groupby(level='user_id').shift(1))
user_launchs.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>launchs</th>
      <th>launchs_yesterday</th>
      <th>lifetime_launchs</th>
    </tr>
    <tr>
      <th>user_id</th>
      <th>event_time</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">100003</th>
      <th>2021-01-18</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2021-01-19</th>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2021-01-20</th>
      <td>2.0</td>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2021-01-21</th>
      <td>0.0</td>
      <td>2.0</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>2021-01-22</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>3.0</td>
    </tr>
  </tbody>
</table>


```python
user_launchs.groupby(level='user_id').cumsum().groupby(['user_id', 'launchs']).cumcount()
```




    user_id  event_time
    100003   2021-01-18    0
             2021-01-19    1
             2021-01-20    0
             2021-01-21    1
             2021-01-22    0
             2021-01-23    0
             2021-01-24    1
             2021-01-25    2
             2021-01-26    0
             2021-01-27    1
    100001   2021-01-18    0
             2021-01-19    0
             2021-01-20    0
             2021-01-21    1
             2021-01-22    2
             2021-01-23    0
             2021-01-24    1
             2021-01-25    0
             2021-01-26    1
             2021-01-27    0
    100002   2021-01-18    0
             2021-01-19    1
             2021-01-20    0
             2021-01-21    0
             2021-01-22    1
             2021-01-23    2
             2021-01-24    3
             2021-01-25    4
             2021-01-26    5
             2021-01-27    6
    dtype: int64



距离上次登录的天数 (`days_since_last_launch`)


```python
user_launchs['days_since_last_launch'] = (user_launchs
                                        .groupby(level='user_id')
                                        .cumsum()
                                        .groupby(['user_id', 'launchs'])
                                        .cumcount()
                                        .groupby(level='user_id').shift(1))
user_launchs.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>launchs</th>
      <th>launchs_yesterday</th>
      <th>lifetime_launchs</th>
      <th>days_since_last_launch</th>
    </tr>
    <tr>
      <th>user_id</th>
      <th>event_time</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">100003</th>
      <th>2021-01-18</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2021-01-19</th>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2021-01-20</th>
      <td>2.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2021-01-21</th>
      <td>0.0</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2021-01-22</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>3.0</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>

类似的，我们还可以滚动的创建 `launchs_last_n_days`


```python
for n in [7, 14]: 
    col = 'launchs_last_{}_days'.format(n)
    user_launchs[col] = (user_launchs
                        .groupby(level='user_id')
                        .launchs
                        .apply(lambda d: d.rolling(n).sum().shift(1)))
```

给空值填充 `0`, 并格式化为 integer 类型


```python
user_launchs.fillna(0).applymap(int)
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>launchs</th>
      <th>launchs_yesterday</th>
      <th>lifetime_launchs</th>
      <th>days_since_last_launch</th>
      <th>launchs_last_7_days</th>
      <th>launchs_last_14_days</th>
    </tr>
    <tr>
      <th>user_id</th>
      <th>event_time</th>
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
      <th rowspan="10" valign="top">100003</th>
      <th>2021-01-18</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-19</th>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-20</th>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-21</th>
      <td>0</td>
      <td>2</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-22</th>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-23</th>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-24</th>
      <td>0</td>
      <td>1</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-25</th>
      <td>0</td>
      <td>0</td>
      <td>5</td>
      <td>1</td>
      <td>5</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-26</th>
      <td>1</td>
      <td>0</td>
      <td>5</td>
      <td>2</td>
      <td>4</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-27</th>
      <td>0</td>
      <td>1</td>
      <td>6</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">100001</th>
      <th>2021-01-18</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-19</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-20</th>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-21</th>
      <td>0</td>
      <td>2</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-22</th>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-23</th>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-24</th>
      <td>0</td>
      <td>1</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-25</th>
      <td>1</td>
      <td>0</td>
      <td>4</td>
      <td>1</td>
      <td>4</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-26</th>
      <td>0</td>
      <td>1</td>
      <td>5</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-27</th>
      <td>1</td>
      <td>0</td>
      <td>5</td>
      <td>1</td>
      <td>4</td>
      <td>0</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">100002</th>
      <th>2021-01-18</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-19</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-20</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-21</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-22</th>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-23</th>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-24</th>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-25</th>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-26</th>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>4</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2021-01-27</th>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>5</td>
      <td>2</td>
      <td>0</td>
    </tr>
  </tbody>
</table>


是不是很神奇


> HowTos 项目 Github 地址： https://github.com/toutiaoio/HowTos