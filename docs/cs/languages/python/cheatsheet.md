## Cheatsheet

- `text.count('word')`：查找单词的出现次数
- `words=set(text)`：把 `text` 变成 `set`，从而执行 `unique` 操作
- `>>> {w[0] for w in words}`：查找所有单词的第一个字母（`{}` 是 `set` 表示）
- `assert <expression>`：和 C 一样，如果 `expression` 是 False 的话就 RE

### 字符串处理

- `lower(s)`：将 `s` 字符串全部变成小写
- `upper(s)`：将 `s` 字符串全部变成大写
- `remove_punctuation(s)`：把字符串里的标点符号全部替换成 `space`
- `split(c)`：通过字符 `c` 将字符串分割成字符串数组，默认 `c = space`