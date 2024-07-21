---
layout: post
title: "为什么 python 不区分全半角"
date: 2024-07-21 18:00 +0800
categories: 计算机
---

> 转自知乎 [Python的变量名为什么区分大小写，但不区分全半角？](https://www.zhihu.com/question/596405042/answer/3568948103)

在使用诸如 print(__name__) 时，ｐｒｉｎｔ(__ｎａｍｅ__) 也能得到完全一样的结果。

![print(__name__)全半角]({{site.base_url}}/image/2024.7.21.whyp.1.png)  
两者结果相同

实际上不只如此哦，在 <https://peps.python.org/pep-0672/#normalizing-identifiers> 所举的例子中，英语中有连词 ﬁ（f 和 i，但是是一个 unicode 码），也会被 python 视为和 fi 等同；xⁿ 会和 xn 等同，也就是 xⁿ = 8，则 print(xn) 也是 8。

为什么呢？这是因为 python 的标识符采用了 Unicode 的 NFKC（Normalization Form Compatibility Composition，兼容等价分解并以标准形态组合）原则进行标准化，解析相关的 Unicode 码，标准化后一致的标识符视为相同标识符。

NFKC 是一种将相似的变体 Unicode 字符映射到同一个字符的标准化方案（例如 Å U+212B 和 Å U+00C5 均视为 Å U+00C5） ，与此类似的还有 NFC, NFD, NFKD，具体可以查看 [wiki：Unicode等价性](https://zh.wikipedia.org/wiki/Unicode%E7%AD%89%E5%83%B9%E6%80%A7)，以及 [从⽅不是方到Unicode正规化NFD, NFC, NFKD, NFKC](https://xobo.org/unicode-normalization-nfd-nfc-nfkd-nfkc/) 和 [Unicode等价性与正规化](https://medium.com/@wanxiao1994/unicode%E7%AD%89%E4%BB%B7%E6%80%A7%E4%B8%8E%E6%AD%A3%E8%A7%84%E5%8C%96-2eb50b343bc1) 这两篇博客。

举例而言，が 在 NFC 和 NFKC 标准化后还是 が（先等价化拆分，后组合）；而在 NFD 和 NFKD 下则是 か 加上浊点 ◌゙（直接等价化拆分）。而 K（Compatibility）则额外加入了更多兼容性映射，诸如全角半角和合并，上下标合并，连字视为独立字等，但并不包含大小写合并（因为“外观”上并不更相似），这也回答了题主的问题。

![NFC 和 NFD 的对比]({{site.base_url}}/image/2024.7.21.whyp.2.png)  
NFC 和 NFD 的对比

这种方式更适合搜索引擎，文本中视觉一样的字符，在搜索时也应该被同时查阅。至于 python 为什么对标识符这么做，我就不得而知了。不过这种规范化在 python 仅仅适用于标识符。将字符串视为标识符的函数（例如getattr）不会执行规范化。

如果你想手动看看这种规范能进行什么样的映射，可以试试在 python 中运行下面的函数哦

```python
from unicodedata import normalize
print(normalize('NFKC', "Ｈｅｌｌｏ，ｗｏｒｌｄ！"))
print(normalize('NFD', "안녕하세요"))
```

参考：  
<https://peps.python.org/pep-0672/#normalizing-identifiers>  
[从⽅不是方到Unicode正规化NFD, NFC, NFKD, NFKC](https://xobo.org/unicode-normalization-nfd-nfc-nfkd-nfkc/)  
[Unicode等价性与正规化](https://medium.com/@wanxiao1994/unicode%E7%AD%89%E4%BB%B7%E6%80%A7%E4%B8%8E%E6%AD%A3%E8%A7%84%E5%8C%96-2eb50b343bc1) 
[wiki：Unicode等价性](https://zh.wikipedia.org/wiki/Unicode%E7%AD%89%E5%83%B9%E6%80%A7)
