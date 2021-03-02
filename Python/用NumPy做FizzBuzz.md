---
title: "Python: 用 NumPy 做 FizzBuzz"
url: https://toutiao.io/posts/kp2tgry
---

写一个程序，输出从 1 到 n 数字的字符串表示。

1. 如果 n 是3的倍数，输出"Fizz"；
2. 如果 n 是5的倍数，输出"Buzz"；
3. 如果 n 同时是3和5的倍数，输出 "FizzBuzz"；
4. 否则输出　n.


```python
import numpy as np
x = np.arange(1, 101)

np.select([(x%3 == 0) & (x%5 == 0), (x%3 == 0), (x%5 == 0)], ['FizzBuzz','Fizz','Buzz'], x) 
```




    array(['1', '2', 'Fizz', '4', 'Buzz', 'Fizz', '7', '8', 'Fizz', 'Buzz',
           '11', 'Fizz', '13', '14', 'FizzBuzz', '16', '17', 'Fizz', '19',
           'Buzz', 'Fizz', '22', '23', 'Fizz', 'Buzz', '26', 'Fizz', '28',
           '29', 'FizzBuzz', '31', '32', 'Fizz', '34', 'Buzz', 'Fizz', '37',
           '38', 'Fizz', 'Buzz', '41', 'Fizz', '43', '44', 'FizzBuzz', '46',
           '47', 'Fizz', '49', 'Buzz', 'Fizz', '52', '53', 'Fizz', 'Buzz',
           '56', 'Fizz', '58', '59', 'FizzBuzz', '61', '62', 'Fizz', '64',
           'Buzz', 'Fizz', '67', '68', 'Fizz', 'Buzz', '71', 'Fizz', '73',
           '74', 'FizzBuzz', '76', '77', 'Fizz', '79', 'Buzz', 'Fizz', '82',
           '83', 'Fizz', 'Buzz', '86', 'Fizz', '88', '89', 'FizzBuzz', '91',
           '92', 'Fizz', '94', 'Buzz', 'Fizz', '97', '98', 'Fizz', 'Buzz'],
          dtype='<U21')


> HowTos 项目 Github 地址： https://github.com/toutiaoio/HowTos
