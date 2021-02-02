---
title: 在 MySQL 中查询时间和日期
---

MySQL 具有以下函数来获取当前日期和时间：

```sql
SELECT now();  -- datetime
SELECT curdate(); --date
SELECT curtime(); --time in 24-hour format
```

要查找两个日期或时间戳之间的数据:

```sql
SELECT *
FROM task_instance
where execution_date between '2021-01-01' and '2021-01-31';

-- 也可以写成时间戳格式
SELECT *
FROM task_instance
WHERE execution_date BETWEEN '2021-01-01 00:00:00' AND '2021-01-31 23:59:59';

SELECT *
FROM task_instance
WHERE execution_date >= '2021-01-01' AND  execution_date < '2021-02-01';
```

查询最近一周的数据:

```sql
SELECT *
FROM task_instance
WHERE execution_date > (now() + interval 1 week);
```

也可以通过函数`DATE_ADD` 和　`DATE_SUB` 来计算时间

```sql
SELECT *
FROM task_instance
WHERE execution_date BETWEEN DATE_SUB(now(), interval 1 week) AND DATE_ADD(now(), interval 1 day);
```

从时间戳里面提取年月日时分秒和星期几等：

```sql
SELECT year(now());
SELECT month(now());
SELECT day(now());
SELECT hour(now());
SELECT minute(now());
SELECT second(now());

SELECT dayofweek(now()); -- 从周日开始周六结束，所以 1 是周日
```

时间戳转成UNIX时间戳

```sql
SELECT unix_timestamp('2021-02-01');
SELECT unix_timestamp('2021-02-01 18:50:00');
SELECT unix_timestamp(); -- 等同于 SELECT unix_timestamp(now());
```

计算两个时间字段之间的间隔

```sql
SELECT unix_timestamp(end_date) - unix_timestamp(start_date)
FROM task_instance;

-- 用 sec_to_time 函数转换成时分秒格式

SELECT sec_to_time(unix_timestamp(end_date) - unix_timestamp(start_date))
FROM task_instance;

-- TIMEDIFF 转换成时分秒格式
SELECT TIMEDIFF(end_date, start_date)
FROM task_instance;

-- 计算间隔天数
SELECT DATEDIFF("2021-02-01", "2020-06-30");
```
