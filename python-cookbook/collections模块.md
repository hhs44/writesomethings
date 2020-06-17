# collections模块

## Counter（）

counter对象可以接受任意的由可哈希的元素构成的序列对象，counter对象类是于一个字典，如下：

```python
Counter({'eyes': 8, 'the': 5, 'look': 4, 'into': 3, 'my': 3, 'around': 2, "you're": 1, "don't": 1, 'under': 1, 'not': 1})
```

包含方法

+ most_common(n)：出现频率最高的字段，n表示出现频率最高的几个

## defaultdict（list/dict/tuple/str）





## namedtuple()

你有一段通过下标访问列表或者元组中元素的代码，但是这样有时候会使得你的 代码难以阅读，于是你想通过名称来访问元素。
```python

>>> from collections import namedtuple 
>>> Subscriber = namedtuple('Subscriber', ['addr', 'joined']) 
>>> sub = Subscriber('jonesy@example.com', '2012-10-19') 
>>> sub
Subscriber(addr='jonesy@example.com', joined='2012-10-19') 
>>> sub.addr
'jonesy@example.com' 
>>> sub.joined 
'2012-10-19'
```

同时也提供了一个修改元组的方法_repalce()



```python
s = s._replace(shares=75)
```

## ChainMap()

映射，多字典

一个 ChainMap 接受多个字典并将它们在逻辑上变为一个字典。然后，这些字典并 不是真的合并在一起了，ChainMap 类只是在内部创建了一个容纳这些字典的列表并重 新定义了一些常见的字典操作来遍历这个列表。大部分字典操作都是可以正常使用的，
比如：

```python
a = {'x': 1, 'z': 3 }

b = {'y': 2, 'z': 4 }
from collections import ChainMap 
c = ChainMap(a,b) 

print(c['x']) # Outputs 1 (from a) 

print(c['y']) # Outputs 2 (from b)
print(c['z']) # Outputs 3 (from a)
```

