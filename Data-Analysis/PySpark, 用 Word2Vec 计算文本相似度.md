---
title: "PySpark: 用 Word2Vec 计算文本相似度"
url: https://toutiao.io/posts/cfpu4ts
---

在这个示例中我们来介绍在 PySpark 中如何通过 Word2Vec (词向量模型) 来计算文本的相似度

Word2Vec顾名思义，这是一个将单词转换成向量形式的工具。通过转换，可以把对文本内容的处理简化为向量空间中的向量运算，计算出向量空间上的相似度，来表示文本语义上的相似度.


```python
import findspark
findspark.init()
```

```python
import pandas as pd
from pyspark.sql import SparkSession
from pyspark.sql.types import (
    StringType,
    ArrayType,
    FloatType
)
from pyspark.sql.functions import (
    udf,
    col
)
from pyspark.ml.feature import Word2Vec
# !pip install jieba
import jieba
```

```python
spark = (SparkSession
    .builder
    .appName("pyspark-word2vec-cosine-similarity")
    .getOrCreate())
```

我们准备了拉取了一份开发者头条上的文章列表作为示例数据


```python
df = (spark.
      read.
      csv('/home/eric/Sync/datasets/misc/tech-posts.csv', header=True)
      .select('id', 'title'))
```


```python
df.show()
```

    +---+------------------------------------+
    | id|                               title|
    +---+------------------------------------+
    |  9|         [PDF]树莓派杂志《MagPi》...|
    | 16|         Python 学习资源列表（kir...|
    |  8|           Scala 最佳实践《Effect...|
    |  1|大众点评网的架构设计与实践（图文版）|
    |  5|             JavaScript 与有限状态机|
    |  6|                    iOS 开发工具列表|
    |  7|                HTML5/JavaScript ...|
    |  2|                    Angular 编程思想|
    | 72|  [译] 亲爱的项目经理，我恨你（外...|
    |  4|      如何让 Python 代码运行得更快？|
    | 73|                如何吸引技术合伙人？|
    | 10|                [PDF]Puppet 入门教程|
    | 75|             先试再问（Matt Ringel）|
    | 11|        如何优化网页转化率？（中篇）|
    | 76|         你喜欢/不喜欢什么编程语言？|
    | 12|     从协作编码到婚礼请柬：GitHub...|
    | 13|           从《寿司之神》学到的5件事|
    | 14|    张小龙2011年在华中科大的演讲实录|
    | 15|                       GitHub 好声音|
    | 96|         在 YC 学到的58件事（Amir...|
    +---+------------------------------------+
    only showing top 20 rows
    


这里我们用到了 [jieba](https://github.com/fxsjy/jieba) 分词工具，把分词封装为一个 udf 对上面的 DataFrame 的 `title` 进行分词


```python
def jieba_seg(x):
    return [w for w in jieba.cut(x) if len(w)>1]
```


```python
jieba_seg_udf = udf(jieba_seg, ArrayType(StringType()))
```


```python
df = df.withColumn('words', jieba_seg_udf(df['title']))
```


```python
df.printSchema()
```

    root
     |-- id: string (nullable = true)
     |-- title: string (nullable = true)
     |-- words: array (nullable = true)
     |    |-- element: string (containsNull = true)
    


拟合 Word2Vec 模型, 输出向量化


```python
model = Word2Vec(numPartitions=10, inputCol='words', outputCol='vecs').fit(df)
```


```python
model.getVectors().count()
```

    2703



模型转换并做交叉JOIN


```python
df_transformed = model.transform(df)
```


```python
df_cross = df_transformed.select(
    col('id').alias('id1'),
    col('vecs').alias('vecs1')).crossJoin(df_transformed.select(
        col('id').alias('id2'),
        col('vecs').alias('vecs2'))
)
```


```python
df_cross.show()
```

    +---+--------------------+---+--------------------+
    |id1|               vecs1|id2|               vecs2|
    +---+--------------------+---+--------------------+
    |  9|[0.00604514013975...|  9|[0.00604514013975...|
    |  9|[0.00604514013975...| 16|[0.00209978688508...|
    |  9|[0.00604514013975...|  8|[-0.0274142913985...|
    |  9|[0.00604514013975...|  1|[-0.0080286560580...|
    |  9|[0.00604514013975...|  5|[0.00677914172410...|
    |  9|[0.00604514013975...|  6|[-0.0022107351881...|
    |  9|[0.00604514013975...|  7|[3.40434722602367...|
    |  9|[0.00604514013975...|  2|[-0.0182423858592...|
    |  9|[0.00604514013975...| 72|[0.00141374469967...|
    |  9|[0.00604514013975...|  4|[-0.0282975720862...|
    |  9|[0.00604514013975...| 73|[-0.0131217022426...|
    |  9|[0.00604514013975...| 10|[0.00280306519319...|
    |  9|[0.00604514013975...| 75|[-9.9342122363547...|
    |  9|[0.00604514013975...| 11|[0.01116290185600...|
    |  9|[0.00604514013975...| 76|[0.01198206515982...|
    |  9|[0.00604514013975...| 12|[-0.0103134274748...|
    |  9|[0.00604514013975...| 13|[7.89615442045033...|
    |  9|[0.00604514013975...| 14|[0.00366029532160...|
    |  9|[0.00604514013975...| 15|[0.00222078245133...|
    |  9|[0.00604514013975...| 96|[-7.1811025263741...|
    +---+--------------------+---+--------------------+
    only showing top 20 rows
    


用 cosine 构建相似度计算 udf


```python
from scipy import spatial

@udf(returnType=FloatType())
def sim(x, y):
    return float(1 - spatial.distance.cosine(x, y))
```

计算两个向量间的相似度 `sim`


```python
df_cross = df_cross.withColumn('sim', sim(df_cross['vecs1'], df_cross['vecs2']))
```

我们用一个id来看一下效果, 按相似度降序排列取最相似的 top 10


```python
test_id = 7445
```


```python
pdf1 = df_cross.filter(col('id1')==test_id).toPandas()
```


```python
sim_top10 = pdf1[pdf1.sim<1].sort_values('sim', ascending=False).head(10)
sim_top10
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id1</th>
      <th>vecs1</th>
      <th>id2</th>
      <th>vecs2</th>
      <th>sim</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>14797</th>
      <td>7445</td>
      <td>[0.06699860654771328, -0.07967841532081366, -0...</td>
      <td>6727</td>
      <td>[0.11702847061678767, -0.10443547181785107, -0...</td>
      <td>0.985086</td>
    </tr>
    <tr>
      <th>5564</th>
      <td>7445</td>
      <td>[0.06699860654771328, -0.07967841532081366, -0...</td>
      <td>13586</td>
      <td>[0.05327895730733872, -0.07372163236141205, -0...</td>
      <td>0.982657</td>
    </tr>
    <tr>
      <th>4920</th>
      <td>7445</td>
      <td>[0.06699860654771328, -0.07967841532081366, -0...</td>
      <td>13061</td>
      <td>[0.07758450843393803, -0.07245811559259892, -0...</td>
      <td>0.981890</td>
    </tr>
    <tr>
      <th>3441</th>
      <td>7445</td>
      <td>[0.06699860654771328, -0.07967841532081366, -0...</td>
      <td>3443</td>
      <td>[0.041676432359963655, -0.04655478027416393, -...</td>
      <td>0.981710</td>
    </tr>
    <tr>
      <th>4264</th>
      <td>7445</td>
      <td>[0.06699860654771328, -0.07967841532081366, -0...</td>
      <td>12563</td>
      <td>[0.04088592156767845, -0.04963424289599061, -0...</td>
      <td>0.981029</td>
    </tr>
    <tr>
      <th>11041</th>
      <td>7445</td>
      <td>[0.06699860654771328, -0.07967841532081366, -0...</td>
      <td>10971</td>
      <td>[0.07418017694726586, -0.08750347793102264, -0...</td>
      <td>0.980928</td>
    </tr>
    <tr>
      <th>14297</th>
      <td>7445</td>
      <td>[0.06699860654771328, -0.07967841532081366, -0...</td>
      <td>6150</td>
      <td>[0.0703863805780808, -0.08529900076488653, -0....</td>
      <td>0.979687</td>
    </tr>
    <tr>
      <th>14950</th>
      <td>7445</td>
      <td>[0.06699860654771328, -0.07967841532081366, -0...</td>
      <td>6892</td>
      <td>[0.044685299907411845, -0.05636310949921608, -...</td>
      <td>0.978825</td>
    </tr>
    <tr>
      <th>1879</th>
      <td>7445</td>
      <td>[0.06699860654771328, -0.07967841532081366, -0...</td>
      <td>1901</td>
      <td>[0.07205328019335866, -0.07813337352126837, -0...</td>
      <td>0.978803</td>
    </tr>
    <tr>
      <th>13691</th>
      <td>7445</td>
      <td>[0.06699860654771328, -0.07967841532081366, -0...</td>
      <td>5431</td>
      <td>[0.06258831359446049, -0.07637270260602236, -0...</td>
      <td>0.978398</td>
    </tr>
  </tbody>
</table>


```python
df.filter(col('id')==test_id).toPandas()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>title</th>
      <th>words</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>7445</td>
      <td>iOS 性能提升总结</td>
      <td>[iOS, 性能, 提升, 总结]</td>
    </tr>
  </tbody>
</table>


```python
df.filter(df.id.isin(sim_top10['id2'].to_list())).toPandas()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>title</th>
      <th>words</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1901</td>
      <td>iOS 性能优化 (chenkai)</td>
      <td>[iOS, 性能, 优化, chenkai]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3443</td>
      <td>一次 TableView 性能优化经历（iOS） (Mr.Yang)</td>
      <td>[一次, TableView, 性能, 优化, 经历, iOS, Mr, Yang]</td>
    </tr>
    <tr>
      <th>2</th>
      <td>12563</td>
      <td>iOS 开发中的 11 种锁以及性能对比</td>
      <td>[iOS, 开发, 11, 种锁, 以及, 性能, 对比]</td>
    </tr>
    <tr>
      <th>3</th>
      <td>13061</td>
      <td>iOS 下的图片处理与性能优化</td>
      <td>[iOS, 图片, 处理, 性能, 优化]</td>
    </tr>
    <tr>
      <th>4</th>
      <td>13586</td>
      <td>iOS 界面性能优化浅析</td>
      <td>[iOS, 界面, 性能, 优化, 浅析]</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10971</td>
      <td>iOS 性能优化探讨</td>
      <td>[iOS, 性能, 优化, 探讨]</td>
    </tr>
    <tr>
      <th>6</th>
      <td>5431</td>
      <td>iOS 开发性能提高</td>
      <td>[iOS, 开发, 性能, 提高]</td>
    </tr>
    <tr>
      <th>7</th>
      <td>6150</td>
      <td>微信读书 iOS 性能优化总结</td>
      <td>[微信, 读书, iOS, 性能, 优化, 总结]</td>
    </tr>
    <tr>
      <th>8</th>
      <td>6727</td>
      <td>渲染性能</td>
      <td>[渲染, 性能]</td>
    </tr>
    <tr>
      <th>9</th>
      <td>6892</td>
      <td>[译] iOS 性能优化：Instruments 工具的救命三招</td>
      <td>[iOS, 性能, 优化, Instruments, 工具, 救命, 三招]</td>
    </tr>
  </tbody>
</table>


```python
spark.stop()
```

这么看效果还是不错的.


> HowTos 项目 Github 地址： https://github.com/toutiaoio/HowTos