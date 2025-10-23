---
layout: post
title: "不同语言字符串对比"
date: 2025-10-23 15:28:00 +0800
categories: 计算机
---

## 表格

**普通 escape string 为 `0-0-0`；raw string、format string、多行字符串分别在三个位置设为 1，例如 `1-0-1` 表示多行 raw string，但不是 format string。防止产生“这个语言明明有多行字符串但为什么是 N/A”的误解**

escape string、raw string、format string 分别简写为 estr、rstr、fstr

另外，多行不考虑

```
s = "123, \
    456"
```

这种形式，因为本质上不是多行字符串：有的语言是用作代码换行，对一般语句也生效（如 c++/python）；有的语言则是字符串内语法，用于删除后续多余空白（如 rust/toml）。因此不考虑

| 语言 | estr | 多行 estr | rstr | 多行 rstr | fstr | 多行 fstr | fstr 插值语法 | frstr | 多行 frstr | 注释 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
|  | `0-0-0` | `0-0-1` | `0-1-0` | `0-1-1` | `1-0-0` | `1-0-1` |  | `1-1-0` | `1-1-1` |  |
| **Bash** | N/A | `$'...'` | N/A | `'...'` | N/A | N/A | `$variable`、`${expression}`、`$((arithmetic-expression))`、`$(command)` 与弃用中的 `` `command` `` | N/A | `"..."` | `'...'` 里不可包含单引号（即使 `\'` 也不行），不过 `$'...'` 和 `"..."` 可以用转义的单双引号 |
| **Python** | `"..."` / `'...'` | `"""..."""` / `'''...'''` | `r"..."` (or `r'...'`, 后略) | `r"""..."""` | `f"..."` | `f"""..."""` | `{expression}` | `fr"..."` (or `rf`, 后略) | `fr"""..."""` | |
| **TypeScript** | `"..."` / `'...'` | N/A | N/A | N/A | N/A | `` `...` `` | `${expression}` | N/A | `` String.raw`...` `` | |
| **Rust** | N/A | `"..."` | N/A | `r#"..."#` (`#` 可以没有或有若干个) | N/A | `format!("...")` (在 `std::fmt` 中，后略) | `format!("...{}...", expression)` | N/A | `format!(r#"..."#)` | 单引号表示 char |
| **C++** | `"..."` | N/A | N/A | N/A | `std::format("...")` (在 `<format>` 中，后略) | N/A | `std::format("...{}...", expression)` | N/A | N/A | 单引号表示 char |
| **JSON** | `"..."` | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | 不支持单引号 |
| **YAML** | N/A | `"..."` (空一行会被理解为空格) | N/A | `'...'`、`>` (空一行会被理解为空格)、`\|` | N/A | N/A | N/A | N/A | N/A | |
| **TOML** | `"..."` | `"""..."""` | `'...'` | `'''...'''` | N/A | N/A | N/A | N/A | N/A | |


## 部分参考资料

Bash

- <https://www.gnu.org/software/bash/manual/bash.html#Quoting-1>
- <https://www.gnu.org/software/bash/manual/bash.html#Shell-Expansions>

Python·

- <https://docs.python.org/3/library/stdtypes.html#text-sequence-type-str>
- <https://docs.python.org/3/tutorial/inputoutput.html#fancier-output-formatting>

YAML

- <https://yaml.org/spec/1.2.2/#73-flow-scalar-styles>·

TOML

- <https://toml.io/cn/v1.0.0#%E5%AD%97%E7%AC%A6%E4%B8%B2>