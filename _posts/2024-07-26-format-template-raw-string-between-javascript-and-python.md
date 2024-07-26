---
layout: post
title: "javascript 和 python 中 format (template)、raw string 等的区别"
date: 2024-07-26 23:27 +0800
categories: 计算机
---

我们来讨论字符串书写中的三点：**raw**, **format (template)** 和**允许回车断行**。

python 中，字符串的 raw 和 format (template) 分别以 `r` 和 `f` 开头；字符串的“允许回车断行”则是使用 `"""` 或 `'''`；

javascript 中，字符串的 template (format) 和“允许回车断行”则都是通过 ``` `` ``` 符实现，raw **仅能** 在 template 的基础上添加 tagfunction `String.raw`。

对比代码如下

python

```python
a = 'Hello'

str_normal = "123; \'; {a};"
print(str_normal)
# 123; '; {a};

str_only_raw = r"123; \'; {a};"
print(str_only_raw)
# 123; \'; {a};
str_only_format = f"123; \'; {a};"
print(str_only_format)
# 123; '; Hello;
str_only_allow_enter = """
123; \'; {a};
"""
print(str_only_allow_enter)
#
# 123; '; {a};
#

str_raw_and_format = rf"123; \'; {a};"
print(str_raw_and_format)
# 123; \'; Hello;
str_raw_and_allow_enter = r"""
123; \'; {a};
"""
print(str_raw_and_allow_enter)
#
# 123; \'; {a};
#
str_format_and_allow_enter = f"""
123; \'; {a};
"""
print(str_format_and_allow_enter)
#
# 123; '; Hello;
#

str_raw_and_format_and_allow_enter = rf"""
123; \'; {a};
"""
print(str_raw_and_format_and_allow_enter)
#
# 123; \'; Hello;
#
```

javascript

```javascript
let a = 'Hello';

let str_normal = "123; \'; ${a};";
console.log(str_normal);
// 123; '; ${a};

// let no_str_only_raw = String.raw`123; \'; ${a};`;    <- still template and allow enter
// console.log(no_str_only_raw);
// 123; \'; ${a};
// let no_str_only_template = `123; \'; ${a};`;         <- still allow enter
// console.log(no_str_only_template)
// 123; '; Hello;
// let no_str_only_allow_enter = `                      <- still template
// 123; \'; ${a};
// `;
// console.log(no_str_only_allow_enter);
//
// 123; '; ${a};
//

// let no_str_only_raw_and_template = String.raw`123; \'; ${a};`;   <- still allow enter
// console.log(str_raw_and_template);
// 123; \'; Hello;
// let no_str_only_raw_and_allow_enter = String.raw`                <- still template
// 123; \'; ${a};
// `;
// console.log(no_str_only_raw_and_allow_enter);
//
// 123; \'; ${a};
//
let str_template_and_allow_enter = `
123; \'; ${a};
`;
console.log(str_template_and_allow_enter);
//
// 123; '; Hello;
//

let str_raw_and_template_and_allow_enter = String.raw`
123; \'; ${a};
`;
console.log(str_raw_and_template_and_allow_enter);
//
// 123; \'; Hello;
//
```

也许是因为“template (format) 和允许回车断行是比 raw 更常见的选项”，但是 javascript 的设计确实比较奇怪。在我想使用的 raw string 时（例如我想存放一个 template (format) string 的使用示例），却不得不处理 template string 本身。无法真正使用独立的 raw string。

“允许回车断行”和其他功能合并是小事，但是独立的 raw string 确实是我期待的函数/语法。
