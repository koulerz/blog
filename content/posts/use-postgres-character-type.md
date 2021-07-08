---
title: "PostgreSQL 中的字符串类型"
date: 2021-07-09T02:52:43+08:00
publishdate: 2021-07-09T02:52:43+08:00
lastmod: 2021-07-09T02:52:43+08:00
draft: false
tags: ["postgresql", "database"]
---

|              | char                                 | varchar    | text   |
| :----------- | :----------------------------------- | :--------- | ------ |
| 是否固定长度 | 是                                   | 否         | 否     |
| 空白字符补全 | 是                                   | 否         | 否     |
| 最大长度限制 | 指定的长度                           | 指定的长度 | 无限制 |
| 默认长度     | char(1)                              | 无限制     | 无限制 |
| 超出范围     | 返回错误(若超出部分是空字符串则截取) | 返回错误   | 无限制 |

- _char, varchar, text 类型之间几乎没有性能差异。char 类型因为填充空白字符效率可能更低_
- _char 类型会在长度不足时以空白字符填补，所以当字符串末尾的空白字符有意义时，可能会产生混淆_
- _任何情况下，PostgreSQL 能被存储的最长的字符串是 1GB，所以在类型中所谓的无限制，最长也不能超过 1GB_
- _text 等同于不限制长度的 carchar, 区别在于 varchar(n) 可以限制长度，数据超出长度时将会返回错误_
- _相对于 text 类型，varchar(n) 明显的好处在于限制长度。但考虑到未来可能更改限制的长度，使用 text 类型并在代码逻辑中限制长度可能是更好的选择_

# 参考

- [PostgreSQL 字符类型](http://www.postgres.cn/docs/13/datatype-character.html)
- [char-varchar-text-性能测试](https://www.depesz.com/2010/03/02/charx-vs-varcharx-vs-varchar-vs-text/)
