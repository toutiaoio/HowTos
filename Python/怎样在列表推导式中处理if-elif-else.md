---
title: Python: 怎样在列表推导式中处理if-elif-else
url: https://toutiao.io/posts/bhb7wnu
---

在列表推导式中用 `if-else` 很简单


```python
print([ f"{i}: 偶数" if i%2 == 0 else f"{i}: 奇数" for i in range(20)])
```

    ['0: 偶数', '1: 奇数', '2: 偶数', '3: 奇数', '4: 偶数', '5: 奇数', '6: 偶数', '7: 奇数', '8: 偶数', '9: 奇数', '10: 偶数', '11: 奇数', '12: 偶数', '13: 奇数', '14: 偶数', '15: 奇数', '16: 偶数', '17: 奇数', '18: 偶数', '19: 奇数']


但是怎么处理 `if-elif-else` 多条件判断呢？　比如我们想用列表推导式来做 FizzBuzz

写一个程序，输出从 1 到 n 数字的字符串表示。

1. 如果 n 是3的倍数，输出"Fizz"；
2. 如果 n 是5的倍数，输出"Buzz"；
3. 如果 n 同时是3和5的倍数，输出 "FizzBuzz"；
4. 否则输出　n.



```python
print(['fizzbuzz' if (i % 5 == 0 and i % 3 == 0)
        else 'fizz' if i % 5 == 0 
        else 'buzz' if i % 3 == 0 
        else i for i in range(1, 100)])
```

    [1, 2, 'buzz', 4, 'fizz', 'buzz', 7, 8, 'buzz', 'fizz', 11, 'buzz', 13, 14, 'fizzbuzz', 16, 17, 'buzz', 19, 'fizz', 'buzz', 22, 23, 'buzz', 'fizz', 26, 'buzz', 28, 29, 'fizzbuzz', 31, 32, 'buzz', 34, 'fizz', 'buzz', 37, 38, 'buzz', 'fizz', 41, 'buzz', 43, 44, 'fizzbuzz', 46, 47, 'buzz', 49, 'fizz', 'buzz', 52, 53, 'buzz', 'fizz', 56, 'buzz', 58, 59, 'fizzbuzz', 61, 62, 'buzz', 64, 'fizz', 'buzz', 67, 68, 'buzz', 'fizz', 71, 'buzz', 73, 74, 'fizzbuzz', 76, 77, 'buzz', 79, 'fizz', 'buzz', 82, 83, 'buzz', 'fizz', 86, 'buzz', 88, 89, 'fizzbuzz', 91, 92, 'buzz', 94, 'fizz', 'buzz', 97, 98, 'buzz']


> HowTos 项目 Github 地址： https://github.com/toutiaoio/HowTos
