# 2022/11/15

## 1 python的map、filter和reduce
配合lambda函数可以取代大部分for循环
### 1.1 map
- 迭代器的每个元素进行map映射
```python
# 字符串和ASCII码的转换
str = "i love python"
res = list(map(lambda x: ord(x), str))
print(res)
# [105, 32, 108, 111, 118, 101, 32, 112, 121, 116, 104, 111, 110]

res = ''.join(map(lambda x: chr(x), res))
print(res)
# i love python
```

### 1.2 filter
- 迭代器的每个元素判断是否保留
```python
# 不及格的挑出来打一顿
score = [11,22,33,44,55,66,77,88,99]
res = filter(lambda x: x<60,score)
print(res)
# <filter object at 0x000001EAB33D5400>
# 返回的是一个filter对象，需要list包装一下
res = list(filter(lambda x: x<60,score))
print(res)
# [11, 22, 33, 44, 55]
```

### 1.3 reduce
- 将迭代器的一个元素和上一次的结果做运算
- 如果不指定初始值，第一次取两个元素
```python
# 列表求和
from functools import reduce
res = reduce(lambda x,y: x+y,[1,2,3,4])
print(res)
# 10
res = reduce(lambda x,y: x+y,[1,2,3,4],10)
print(res)
# 20
```

### 1.4 lambda
- 匿名函数
 ```python
 # 统一修改dict的键值
 # 通过tuple列表的方式来返回多个参数
 info = {
    "aa":11,
    "bb":22,
    "cc":33,
}
res = dict(map(lambda x:('t_'+x, 2*info[x]), info))
print(res)
# {'t_aa': 22, 't_bb': 44, 't_cc': 66}
 ```

## 2 1 js中的map、filter和reduce
配合arrow函数可以取代大部分for循环
### 2.1 map
