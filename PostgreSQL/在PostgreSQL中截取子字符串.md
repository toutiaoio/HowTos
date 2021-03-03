---
title: 在 PostgreSQL 中截取子字符串
url: https://toutiao.io/posts/eov94gx
---

## 截取子串的函数, substring `SUBSTRING(string, start_position, length)`

函数包含三个参数：

- `string`: 字符串字段
- `start_position`: 开始位置
- `length`: 从开始位置截取的长度，该参数为可选，不填的时候为开始位置至末尾.

例如:

```
postgres=# SELECT SUBSTRING ('PostgreSQL', 1, 8);
 substring 
-----------
 PostgreS
(1 row)

postgres=# SELECT SUBSTRING ('PostgreSQL', 8);
 substring 
-----------
 SQL
(1 row)
```

但有时候开始位置并不是固定的，比如我们需要提取出 Email 中的域名，这时我们需要用
到另一个函数 position 来定位出开始位置在哪里

## 定位 position `POSITION(substring in string)`

例子:

获取 Email 地址中 @ 的位置

```
postgres=# SELECT POSITION('@' IN 'tt@toutiao.io');
 position 
 ----------
         3
```

从 Email 中截取出域名

```
postgres=# SELECT SUBSTRING('tt@toutiao.io' FROM POSITION('@' IN
'tt@toutiao.io')+1);
 substring  
 ------------
  toutiao.io
  (1 row)
```

按域名分组统计 Email 地址数量

```SQL
SELECT lower(substring(email                                                                                                                FROM position('@' IN email)+1)) AS domain_name,
       count(id) AS cnt
FROM users
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;


 domain_name |  cnt   
-------------+--------
 qq.com      | 10000000
 gmail.com   | 2900000
 163.com     | 1000000
 126.com     | 1000000
 outlook.com | 1000000
 hotmail.com | 1000000
 foxmail.com | 1000000
 sina.com    | 1000000
 yeah.net    | 1000000
 live.com    | 1000000
```

> HowTos 项目 Github 地址： https://github.com/toutiaoio/HowTos