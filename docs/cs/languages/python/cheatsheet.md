# Cheatsheet

## 基础语法

- `assert <expression>`：和 C 一样，如果 `expression` 是 False 的话就 RE
- `map(function, list)`：把 `list` 中的元素，按照 `function` 一一映射成另一个 `list`.



## 基本数据类型

### 字符串/文本处理

- `lower(s)`：将 `s` 字符串全部变成小写
- `upper(s)`：将 `s` 字符串全部变成大写
- `swapcase(s)`：将 `s` 大小写交换
- `remove_punctuation(s)`：把字符串里的标点符号全部替换成 `space`
- `split(c)`：通过字符 `c` 将字符串分割成字符串数组，默认 `c = space`
- `ord(c)`: 查看 ASCII 值
- `text.count('word')`：查找单词的出现次数



### Dictionary

```python
>>> d = {'I': 1, 'V': 5, 'X': 10}
>>> d['I']
>>> d.keys() # 索引列表
>>> d.values() # 值列表
>>> d.items() # 返回 pair 列表
>>> 'X' in d # output: True
>>> d.get('X', 0) # 如果 X 有值，那么就返回值；否则返回后面的 default 值
>>> {x: x*x for x in range(1, 10)} # 一下定义了十个值
```



### Set

```python
>>> words=set(text) #把 `text` 变成 `set`，从而执行 `unique` 操作
>>> {w[0] for w in words} #查找所有单词的第一个字母（`{}` 是 `set` 表示）
```

