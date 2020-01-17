---
title: "PostgreSQL 中 Json 和 Jsonb 的区别"
date: 2016-04-02T15:47:06+08:00
publishdate: 2016-04-02
lastmod: 2020-01-17
draft: false
tags: ["postgresql", "database"]
---
Postgres 中的 Json 和 Jsonb 数据类型都是用于储存 JSON ( JavaScript Object Notation ) 格式数据。虽然 Text 数据类型也可以用来储存 JSON 数据， 但 JSON 数据类型的优势在于它会根据 JSON 规则来强制要求每个被储存的值都是合法的 JSON 数据。

一般情况下， 除非有特别的要求（比如针对对象键排列顺序的遗留假设，legacy assumption）， 否则的话， 大多数应用程序都应该优先使用 jsonb 类型来储存 JSON 数据。

---

## Json 数据类型：

json 数据类型储存输入文本的精确拷贝，处理函数在每次执行的时候，都必须对这些文本重新进行分析。

json 数据类型会保留文本中与 Json 语义完全无关的空白字符，各个键在 Json 对象内的排列顺序以及具有相同键的值。


## Jsonb 数据类型：

Jsonb 数据类型以无压缩（decomposed）二进制格式来储存数据，因为格式转换带来的花销，这种类型在处理输入的时候速度会稍微慢一些，但是因为这种类型的数据并不需要重新进行分析，所以这种数据的处理速度会明显地快很多。

jsonb 支持索引特性，这是一个明显的优点。

jsonb 不会保留任何无关的空白，不会保留对象键的排列顺序，也不会保留任何重复的对象键。如果输入里面指定了重复的键，那么只有最后一个值会被保留。

jsonb 类型可以检测一个 jsonb 值是否包含了另一个 jsonb 值，而 json 类型并不具备这样的特性。

jsonb 会拒绝那些超出 PostgreSQL 数字类型范围的数字，而 json 则不会这样做。

