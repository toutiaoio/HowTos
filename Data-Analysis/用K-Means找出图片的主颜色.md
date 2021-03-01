---
title: 用K-Means找出图片的主颜色
---

### 逻辑

我们先把图片转换成 RGB 矩阵，然后我们通过 K-Means 聚类，每个聚类的中心我们就可以认为是图片的主色。

### 加载图片

我们将使用matplotlib.image加载图像，然后通过遍历图像像素创建红、绿、蓝三色的一个DataFrame。

```python
%matplotlib inline 
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import matplotlib.image as img
from scipy.cluster.vq import kmeans, vq

sns.set(rc={"figure.figsize": (16, 9)})
```

```python
image = img.imread("./data/flower.jpg")
image.shape
```

    (1080, 1080, 3)


```python
plt.imshow(image)
```
    <matplotlib.image.AxesImage at 0x7f6cfaabcf90>

![png](https://img.toutiao.io/attachment/c73c644310ae4bd28c90cd48d366688d/w600)


```python
r = []
g = []
b = []
 
for row in image:
    for pixel in row:
        # A pixel contains RGB values
        r.append(pixel[0])
        g.append(pixel[1])
        b.append(pixel[2])
 
df = pd.DataFrame({'red':r, 'green':g, 'blue':b})
 
df.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>red</th>
      <th>green</th>
      <th>blue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>207</td>
      <td>215</td>
      <td>202</td>
    </tr>
    <tr>
      <th>1</th>
      <td>205</td>
      <td>213</td>
      <td>200</td>
    </tr>
    <tr>
      <th>2</th>
      <td>202</td>
      <td>210</td>
      <td>197</td>
    </tr>
    <tr>
      <th>3</th>
      <td>202</td>
      <td>210</td>
      <td>197</td>
    </tr>
    <tr>
      <th>4</th>
      <td>202</td>
      <td>210</td>
      <td>197</td>
    </tr>
  </tbody>
</table>

现在我们可以用肘部法则来找到聚类K的个数


```python
distortions = []
num_clusters = range(1, 7)
 
# Create a list of distortions from the kmeans function
for i in num_clusters:
    cluster_centers, distortion = kmeans(df[['red','green','blue']].values.astype(float), i)
    distortions.append(distortion)
 
# Create a data frame with two lists - num_clusters, distortions
elbow_plot = pd.DataFrame({'num_clusters': num_clusters, 'distortions': distortions})
 
# Creat a line plot of num_clusters and distortions
sns.lineplot(x='num_clusters', y='distortions', data = elbow_plot)
plt.xticks(num_clusters)
plt.show()
```

![png](https://img.toutiao.io/attachment/702420a3d17e4071a30a954d3d9533b9/w600)

现在我们可以看到 `K = 2` 比较合适

### K-Means 和 主颜色

主颜色即聚类的中心

```python
cluster_centers, _ = kmeans(df[['red','green','blue']].values.astype(float), 2)
cluster_centers
```

    array([[ 44.22786005,  73.43192879,  27.9358564 ],
           [198.48993822, 212.59568689, 195.00483147]])

我们需要将它们转成为1×k×3，其中k是簇的数量。并显示出来。


```python
plt.imshow(cluster_centers.reshape(1,2,3)/255.)
```

    <matplotlib.image.AxesImage at 0x7f6cedd08d50>

![file](https://img.toutiao.io/attachment/500bbab0f0f54eebb1432489d02b62e0/w600)

```python
plt.imshow(image)
```

    <matplotlib.image.AxesImage at 0x7f6cedc15310>


![file](https://img.toutiao.io/attachment/dd3d7d41ffc1467ab2fbab73435d63fd/w600)


如果我们现在要给每个像素打上聚类标签，我们可以通过 `scipy.cluster.vq` 中的 `vq` 方法


```python
# 打上标签
df['clusters'] = vq(df, cluster_centers)[0]
df.head()
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>red</th>
      <th>green</th>
      <th>blue</th>
      <th>clusters</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>207</td>
      <td>215</td>
      <td>202</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>205</td>
      <td>213</td>
      <td>200</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>202</td>
      <td>210</td>
      <td>197</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>202</td>
      <td>210</td>
      <td>197</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>202</td>
      <td>210</td>
      <td>197</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>
