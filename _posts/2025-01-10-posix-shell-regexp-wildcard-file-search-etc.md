---
layout: post
title: "POSIX Shell、正则、通配符、文件搜索等"
date: 2024-12-03 21:24:00 +0800
edit: 2025-03-08 01:04:00 +0800
categories: 计算机
---

## 一、前置概念

> 需要学习的内容：PCRE, ERE, BRE, 通配符

### i. shell 兼容性

POSIX shell、Bourne Shell (`sh`) compatible、Unix shell

- Unix shell：概念最广，任何 Unix 上的 shell 都可以称为 Unix shell（例如 `sh`, `bash`, `zsh`, `csh`, `fish`）。
- POSIX shell：符合 POSIX 规范中对 shell 的规定，这样的 shell 称为 POSIX shell（例如 `sh`, `bash`, `zsh`；但 `csh` `fish` 不遵循）。
- Bourne Shell (`sh`) compatible：兼容 `sh` 语法的 shell（例如 `sh`, `bash`, `zsh`；但 `csh` `fish` 不兼容），与 POSIX shell 类似（因为 POSIX 规范的 shell 规则基于原始 `sh`），因此它基本上和 POSIX shell 等价。但是 POSIX shell 的指代更加明确，因此一般会考虑 shell 是否遵循 POSIX。

### ii. 正则表达式

正则表达式、正则 (regular expression, regexp, regex, RE, re)，是一个用于字符串模式匹配的表述规则，在各个编程语言以及各个软件中都十分常用。POSIX 有 Basic RE (BRE) 和 Extended RE (ERE) 两套标准，现代正则一般采用 PCRE 标准，以及其相关的各种兼容性衍生标准（Python, Java, JS 等）。

功能丰富度为 PCRE > ERE >= BRE。

如下是学习网站

- 正则学习：[RegexBuddy: Regular Expressions Quick Start（通用标准 \[PCRE 及其衍生标准\] 教程）](https://www.regular-expressions.info/quickstart.html)，以及这个网站的其他相关网页
- 正则学习：[wikipedia: PCRE 表达式全集](https://zh.wikipedia.org/zh-cn/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F#PCRE%E8%A1%A8%E8%BE%BE%E5%BC%8F%E5%85%A8%E9%9B%86)
- 正则学习：[MRCODE-BOOK: 正则有哪些常见的流派及其特性？](https://zq99299.github.io/note-book/regular/03/03.html)
- 参考：[RegexBuddy: POSIX BRE 和 ERE 的特征](https://www.regular-expressions.info/posix.html)
- 参考：[wikibooks: BRE 和 ERE 的主要区别](https://en.wikibooks.org/wiki/Regular_Expressions/POSIX-Extended_Regular_Expressions)

其中，BRE 中**部分**元字符 (metacharacter) `"()"`、`"{}"`、`"+"`、`"?"`、`"|"` 需要转义，也就是直接输入 `"a{2}"` 会匹配真正的 `"a{2}"` 文本，只有 `"a\{2\}"` 会匹配 `"aa"`；但是另一些元字符 `"[]"`、`"."`、`"*"` 则不需要转义，例如 `"[ab]c"` 可以直接匹配 `"ac"` 与 `"bc"`。而 ERE 基本上都不需要转义，和 PCRE 更类似。

### iii. 通配符

通配符 (wildcard) **不是正则表达式**。尽管它也使用 `? * []` 这些和正则很类似的符号，但它和正则的含义不同。

具体可以参考 [GNU/Linux Command-Line Tools Summary: Wildcards](https://tldp.org/LDP/GNU-Linux-Tools-Summary/html/x11655.htm)。

例如，

- `.` 是一个正则元符号，但不是一个通配符。
- `*` 在正则中代表将**前一个**符号重复 >=0 次；而在通配符中代表 >=0 长度的**任何**符号。粗略类似正则的 `".*"`（注意 `".*"` 一般不匹配换行，虽然通配符中一般也没有换行），考虑换行可以类似正则的 `"[[:print:]]*"` (BRE, ERE, PCRE) 或 `"(\S|\s)*"` (ERE, PCRE)、`"[\S\s]*"` (PCRE)。
- `?` 在正则中代表将前一个符号重复 0/1 次；而在通配符中代表任何一个单独字符。粗略类似正则的 `"."`，同样考虑换行可以类似正则的 `"[[:print:]]"` (BRE, ERE, PCRE) 或 `"(\S|\s)"` (ERE, PCRE)、`"[\S\s]"` (PCRE)。
- `[]` 和正则几乎相同。但是反选形如 `[!abc]`，而正则形如 `[^abc]`。

## 二、搜索处理相关命令

主要介绍 `grep`、`find`、`sed`、`awk`。

### i. `grep`

匹配**部分文本内容**，默认 BRE 规则，

```shell
# 文件形式
grep "BRE规则" "文件1" "文件2" ... "文件n"

# 文件夹形式（依次打开匹配，并非匹配文件名）
grep -r "BRE规则" "文件夹1" "文件夹2" ... "文件夹n"  # -r / -R / --recursive，或者
rgrep "BRE规则" "文件夹1" "文件夹2" ... "文件夹n"

# 管道形式
(standard input pipe) | grep "BRE规则"  # 例如
ls -lah | grep "\.py"
```

`-i` 无视大小写。

注意以上内容均是匹配文件内容（而非文件名），如果匹配文件名，一个做法是使用 ls 列举文件并从管道交给 grep，但更建议使用后文的 `find`。

默认是 BRE 规则，因此如果想要使用 ERE、PCRE、纯文本，需要使用如下命令，

```shell
# ERE (--extended-regexp)
grep -E "ERE规则" "文件1" "文件2" ... "文件n"  # -E / --extended-regexp，或者
egrep "ERE规则" "文件1" "文件2" ... "文件n"

# PCRE (--perl-regexp)
grep -P "PCRE规则" "文件1" "文件2" ... "文件n"  # -P / --perl-regexp

# 纯文本 (--fixed-strings)
grep -F "纯文本" "文件1" "文件2" ... "文件n"  # -F / --fixed-strings，或者
fgrep "纯文本" "文件1" "文件2" ... "文件n"
```

其他选项可使用 `man grep` 自行阅读。

### ii. `find`

匹配**整体文件名**，默认通配符，

```shell
# 列举所有文件
find "文件夹1" "文件夹2" ... "文件夹n"

# 匹配
find "文件夹1" "文件夹2" ... "文件夹n" -name "通配符"
```

`-iname` 无视大小写。

和 `grep "pattern" "file" ...` 不同的是，`find` 是 `find "dir" ... -name "pattern"`，pattern 是一个 key-arg 而不是 position-arg，并且 `-name` 等其他过滤条件要放在最后（其语法规定）。

如果想要使用 ERE，需要使用如下命令 `-regex` 替代 `-name`，但是注意 `-regex` 匹配**整体路径名**而非整体文件名，**单个文件甚至会包括开头的 `./`**，

```shell
# ERE
find "文件夹1" "文件夹2" ... "文件夹n" -regex "ERE规则"
```

`-iregex` 无视大小写。

如果想要使用其他规则，需要额外指定 `-regextype`，根据手册使用 `find -regextype help` 查看有如下几种

```plaintext
'findutils-default', 'awk', 'egrep', 'ed', 'emacs', 'gnu-awk', 'grep', 'posix-awk', 'posix-basic', 'posix-egrep', 'posix-extended', 'posix-minimal-basic', 'sed'
```

```shell
# 其他规则
find "文件夹1" "文件夹2" ... "文件夹n" -regextype "规则类型" -regex "ERE规则"
```

同时支持对规则的否定、合取、析取，并使用转义的括号包裹（当然也要放最后）

```shell
# 举例，-not 可以用 \! 替代
find "文件夹1" "文件夹2" ... "文件夹n" \( -name "通配符1" -or -name "通配符2" \) -and -not -user tom
```

如果要固定搜索深度，使用 `-maxdepth 深度` 和 `-mindepth 深度`。**注意请不要用 `-depth`，它的意思是由深处开始向上搜索（先搜叶子节点再搜亲节点）**。

```shell
# 固定深度
find "文件夹1" "文件夹2" ... "文件夹n" -name "通配符" -maxdepth 3 -mindepth 2
```

除此之外，`find` 还可以用其他性质进行搜索（例如时间、作者等）。可使用 `man find` 自行阅读。

### iii. `awk`

处理文本内容（提取性），TBD

### iv. `sed`

处理文本内容（编辑性），TBD

## 三、POSIX shell 相关命令

### i.引号

> 单引号扩起的字符串不进行转义，而双引号会

```shell
a="hello"
echo ${a}
# hello
echo "${a}"
# hello
echo '${a}'
# ${a}
```

### ii.变量存在性

参考 [shell 编程：冒号 后面跟 等号，加号，减号，问号的意义](https://handerfly.github.io/shell/2019/04/03/shell%E7%BC%96%E7%A8%8B%E5%86%92%E5%8F%B7%E5%8A%A0-%E7%AD%89%E5%8F%B7-%E5%8A%A0%E5%8F%B7-%E5%87%8F%E5%8F%B7-%E9%97%AE%E5%8F%B7/)

```shell
echo ${a:-string}  # == ENV_DICT.get(a, "string")
echo ${a:=string}  # == ENV_DICT.setdefault(a, "string")
echo ${a:+string}  # == if (ENV_DICT.exist(a)) then { ENV_DICT[a] = "string"; "string" } else { "" }
echo ${a:?string}  # == try: ENV_DICT[a]; except: raise "string"
```

### iii.变量修改

```shell
a="abcdefg-abcdefgh"

echo ${a}
# abcdefg-abcdefgh
```

截取

```shell
echo ${a:2}  # 从n1开始截取
# cdefg-abcdefgh

echo ${a:2:7}  # 从n1开始截取n2长度
# cdefg-a
```

删除

\# 与 ## 从前删除，% 与 %% 从后删除，## 和 %% 在使用 * 时进行最大匹配。

```shell
echo ${a#abc}
# defg-abcdefgh

echo ${a##abc}
# defg-abcdefgh

echo ${a#a*d}
# efg-abcdefgh

echo ${a##a*d}
# efgh
```
