---
title: "Python学习进阶2"
date: 2017-08-30T01:37:56+08:00
lastmod: 2017-08-30T01:37:56+08:00
draft: false
tags: ["Python"]
categories: ["Python"]
author: "Liron"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---

####  如何在列表、字典、集合中根据条件筛选数据
##### filter
Python内建的filter()函数用于过滤序列。

和map()类似，filter()也接收一个函数和一个序列。和map()不同的是，filter()把传入的函数依次作用于每个元素，然后根据返回值是True还是False决定保留还是丢弃该元素。

例如，在一个list中，删掉偶数，只保留奇数，可以这么写：

```
def is_odd(n):
    return n % 2 == 1

list(filter(is_odd, [1, 2, 4, 5, 6, 9, 10, 15]))
# 结果: [1, 5, 9, 15]
```

也可以使用列表生成式：

```
[x for x in data if x >= 0]
```

那么使用哪种方式好呢？ 我们可以用`timeit`查看运行的所需时间

```
In [26]: timeit list(filter(lambda x: x >= 0, data))
100000 loops, best of 3: 2.08 µs per loop

In [27]: timeit  [x for x in data if x >= 0]
The slowest run took 4.87 times longer than the fastest. This could mean that an intermediate result is being cached.
1000000 loops, best of 3: 729 ns per loop
```

***结论：***
用列表生成式要快于`filter()`

##### 字典解析

```
d = {x : randint(60, 100) for x in range(20)}
 
 {k: v for k, v in d.items() if v > 90}
```

##### 集合解析

```
In [39]: data
Out[39]: [-4, 2, -9, -2, -9, 1, 3, -2, -7, -6]

In [40]: s  = set(data)

In [41]: s
Out[41]: {-9, -7, -6, -4, -2, 1, 2, 3}

In [42]: {x for x in s if x > 0}
Out[42]: {1, 2, 3}
```

#### 如何为元祖中的每个元素命名，提高程序的可读性
##### 使用常量定义元祖的索引

```
In [50]: NAME, AGE, SEX, EMAIL = range(4)

In [51]: student = ('wangnima', 40, 'male', 'baozou@nima.com')

In [52]: student[NAME]
Out[52]: 'wangnima'

In [53]:
```

##### 使用namedtuple类的形式定义元祖索引

```

In [53]: from collections import namedtuple

In [54]: Student = namedtuple('Student', ['name', 'age', 'sex', 'email'])

In [55]: s = Student('wangnima', 40, 'male', 'baozou@nima.com')

In [56]: s.name
Out[56]: 'wangnima'

In [57]: s.age
Out[57]: 40

In [58]: isinstance(s, tuple)
Out[58]: True
```

#### 2-3 如何统计序列中元素出现的频率

##### 实际案例
1.某随机序列[23,1,2,23,1,2,23,56,7 ...]中，找到出现次数最高的3个元素，它们出现次数是多少？

方法1：

```
In [67]: data = [randint(0, 20) for _ in range(30)]       
                                                          
In [68]: c = dict.fromkeys(data, 0)                       
                                                          
In [69]: for x in data:                                   
    ...:     c[x] += 1                                    
    ...:                                                  
                                                          
In [70]: c                                                
Out[70]:                                                  
{0: 3,                                                    
 2: 1,                                                    
 3: 1,                                                    
 4: 1,                                                    
 6: 2,                                                    
 7: 2,                                                    
 8: 1,                                                    
 10: 2,                                                   
 12: 2,                                                   
 13: 2,                                                   
 14: 1,                                                   
 15: 1,                                                   
 16: 4,                                                   
 17: 4,                                                   
 18: 1,                                                   
 19: 2}                                                   
```
 方法2：
在`python`中有专门处理这类问题的类`Counter`

```
In [71]: from collections import Counter

In [72]: c2 = Counter(data)

In [73]: c2
Out[73]:
Counter({0: 3,
         2: 1,
         3: 1,
         4: 1,
         6: 2,
         7: 2,
         8: 1,
         10: 2,
         12: 2,
         13: 2,
         14: 1,
         15: 1,
         16: 4,
         17: 4,
         18: 1,
         19: 2})

In [74]: c2.most_common(3)
Out[74]: [(16, 4), (17, 4), (0, 3)]
```

#### 2-4 如何根据字典中值的大小，对字典中的项排序

方法1：

```
In [108]: d = {x: randint(60, 100) for x in 'xyzabc'}

In [109]: v = list(d.values())

In [110]: k = list(d.keys())

In [111]: list(zip(v, k))
Out[111]: [(91, 'x'), (99, 'y'), (84, 'z'), (73, 'a'), (63, 'b'), (75, 'c')]

In [112]: sorted(list(zip(v, k)))
Out[112]: [(63, 'b'), (73, 'a'), (75, 'c'), (84, 'z'), (91, 'x'), (99, 'y')]
```

方法2：

```
In [114]: sorted(d.items(), key=lambda x: x[1])
Out[114]: [('b', 63), ('a', 73), ('c', 75), ('z', 84), ('x', 91), ('y', 99)]
```

#### 2-5 如何快速找到多个字典中公共键

要点：转成集合然后取并集

```
In [184]: reduce(lambda a,b : a & b, map(set, map(dict.keys, [{x: randint(1,3) for x in sample('qwertyuio', randint(3, 5))} for _ in range(3)])))
Out[184]: {'q'}
```

#### 2-6 如何让字典保持有序
```
In [187]: from collections import OrderedDict

In [188]: d = OrderedDict()

In [189]: d['a'] = (1, 3)

In [190]: d['b'] = (2, 3)

In [191]: d['c'] = (3, 3)

In [192]: iter(d)
Out[192]: <odict_iterator at 0x19fba7e1c50>

In [193]: list(iter(d))
Out[193]: ['a', 'b', 'c']

In [194]: list(iter(d))
Out[194]: ['a', 'b', 'c']
```

#### 2-7 deque队列
```
In [195]: from collections import deque

In [196]: d = deque([], 5)

In [197]: d.append(randint(1,500))
```
