Title: Python3标准库_collections
Date: 2016-04-4
Category: learning
Tags: python, collections

## 命名元组namedtuple

命名元组顾名思义就是有名称的元组, 其本质是一个继承自tuple的对象, 使用`namedtuple`函数可以快速创建该对象的子类, 并具有一些常用的方法, 设置`verbose=True`可以看到子类的细节;

```python
from collections import namedtuple

Point = namedtuple('Point', ['x', 'y'], verbose=True)
```

命名元组继承了元组非常重要的特性: 不可改变; 和元组一样具有某些场景下作为不可改变的对象使用, 如由于**hashable**特性作为字典的键使用;

由于命名元组的对象特性, 所以访问命名元组中的元素, 除了和元组一样使用下标访问之外, 还可以通过属性名称来访问, 但是却不能和字典一样使用名称下标访问, 这对命名元组的名称提出了一个基本要求: 命名元组的名称必须是合法变量名;

命名元组除了不可更改之外, 其内置的一些方法使其在处理一些数据(尤其是json)时相比一般的对象更加方便, 相比字典又具有安全和hashable的优势;

命名元组的不可更改并不是绝对的, 通过`_replace`方法可以真实修改其中的属性(并不会新建对象);

namedtuple常见的使用方式:

```python
from collections import namedtuple

Point = namedtuple('Point', ['x', 'y'])

# 使用参数创建
point1 = Point(1, 2)
point2 = Point(x=1, y=2)

# 通过列表创建
a = [1, 2]
point3 = Point._make(a)

# 通过字典创建
d = {'x': 1, 'y': 2}
point4 = Point(**d)

# 转换为字典(返回OrderedDict对象)
point1._asdict()

# 修改其中的元素值, 此时对象id并没有发生改变
point1._replace(y=33)
```

## 带默认值的字典defaultdict

常规字典对象并不能对未初始化的键进行更改, 于是在很多场合就需要事先做一次初始化或者检查键是否存在:

```python
from collections import defualtdict

# 常规方法, 设置初始值
d = {}
for key in key_list:
    d.setdefault(k, 0)
for key in key_list:
    d[key] += 1

# 常规方法, 检查键
d = {}
for key in key_list:
    try:
        d[key] += 1
    except KeyError:
        d[key] = 0
        
# 使用defaultdict
d = defaultdict(int)
for key in key_list:
    d[key] += 1    
```

defaultdict更加简洁, 并且高效; 

该函数接收一个`callable`对象, 并在发现键值找不到时, 使用该`callable`对象去处理`__missing__`方法, 从而到达给字典设置默认值的效果;

常见的用法有:

```python
from collections import defaultdict

d1 = defaultdict(int)
d1['a'] += 1

d2 = defaultdict(list)
d2['a'].append(1)

d3 = defaultdict(set)
d3['a'].add(1)

d4 = defualtdict(lambda x: '<missing>')
print(d4['a'])
```

直接操作一个多层字典往往会引起`KeyError`异常, 通过一个递归函数创建的`defaultdict`可以实现多层字典:

```python
from collections import defaultdict

def deep_dict():
    return defaultdict(deep_dict)


if __name__ == '__main__':
    d = deep_dict()
    d['A']['B']['C'] = 'D'
```

有时候并不需要在创建时就设置默认值, 或者得到是从json等结构中获取到字典, 需要在访问的时候设置默认值, 此时可以使用dict的`get()`方法设置默认值, 如:

```python
for json_list in json_dict.get('list', {}):
    for json_item in json_list.get('item', []):
        for value in json_item.get('key', ''):
            print(value)
```

## 顺序字典OrderedDict

默认的字典类型是无序存储的, 如果想要按照一定的顺序存储数组, 就需要用到OrderedDict, 但是需要注意的是, 由于顺序的加入, OrderedDict消耗的内存可能是普通字典的好几倍, 在较大数据量时应该避免使用; 

```python
from collections import OrderedDict

d = OrderedDict()
d['a'] = 1
d['b'] = 2
```

注意: 只有在实例化之后, 按顺序添加的键值对才保持顺序, 在初始化时传入的字典仍然是不保持顺序的。