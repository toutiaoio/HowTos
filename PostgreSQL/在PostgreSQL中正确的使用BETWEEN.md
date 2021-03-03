---
title: 在 PostgreSQL 中正确的使用BETWEEN
url: https://toutiao.io/posts/2rum3hb
---

在 PostgreSQL 中用 `BETWEEN` 查询时间撮(timestamp)　可要小心了，搞不好漏掉一整天的数据

比如我们想查询 2020-12 月有多少新用户注册

```sql
SELECT count(*) user_cnt
FROM users
WHERE created_at BETWEEN '2020-12-01' AND '2020-12-31';
```

实际上查询漏掉了 `2020-12-31` 这天一整天的数据, 上面查询等同于

```sql
SELECT count(*) user_cnt
FROM users
WHERE created_at >= '2020-12-01 00:00:00.000000' AND created_at <= '2020-12-31';
```

所以我们可以写成左闭右开这样来避免这个问题：

```sql
SELECT count(*) user_cnt
FROM users
WHERE created_at >= '2020-12-01' AND created_at < '2021-01-01';
```

> HowTos 项目 Github 地址： https://github.com/toutiaoio/HowTos