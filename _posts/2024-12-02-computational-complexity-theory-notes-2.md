---
layout: post
title: "计算复杂性理论（二）"
date: 2024-12-02 21:04:25 +0800
categories: 计算机
---

<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    jax: ["input/TeX","input/MathML","output/SVG", "output/CommonHTML"],
extensions: ["tex2jax.js","mml2jax.js","MathMenu.js","MathZoom.js", "CHTML-preview.js"],
TeX: {
  extensions: ["AMSmath.js","AMSsymbols.js","noErrors.js","noUndefined.js"]
},
  tex2jax: {
      inlineMath: [ ['$','$'], ["\\(","\\)"] ],
      displayMath: [ ['$$','$$'], ["\\[","\\]"] ],
      processEscapes: true,
      processEnvironments: true
    },
    "HTML-CSS": { availableFonts: ["TeX"] }
  });
</script>
<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML-full"></script>

在上文中，我们介绍了什么是图灵机、语言，以及语言的可判定性．除了直接构造处理语言的图灵机，我们是否还有别的方法，更快地了解一个语言是否可以判定呢？

不知大家有没有听过 [quine](https://en.wikipedia.org/wiki/Quine_(computing)) 程序，这是一种打印自身代码的代码．另外一个有趣的问题是，我们能否斩钉截铁地证明这样的程序一定存在，甚至证明一个比它更强的结论？答案是可以的．

## 二、归约与递归定理

### 2.1 神谕图灵机

**<font color=dodgerblue>定义 2.1</font>** 神谕图灵机 (oracle turing machine) 是这样一台图灵机：对于一个普通的图灵机 $\text{M}$ 和一个语言 $L$，如果 $\text{M}$ 在工作时可以直接询问“掌管 $L$ 的神”，以得知任何一个 $x$ 是否属于 $L$．则称这台图灵机 $\text{M}$ 是以 $L$ 为神谕的图灵机，记作 $\text{M}^L$．

如果一个 $L$ 是原本不可判定的语言，那么 $\text{M}^L$ 相比普通图灵机就能判定更多语言了．我们可以用 $\text{M}^L$ 引出对不可判定语言的另一种判断方式．

### 2.2 图灵归约

**<font color=dodgerblue>定义 2.2</font>** 对于语言 $A, B$，如果存在神谕图灵机 $\text{M}^B$ 判定语言 $A$，则称 $A$ 语言可以图灵归约 (turing reduction) 到 $B$ 语言，记作 $A \leqslant_T B$．

$\text{M}^B$ 判定语言 $A$，也即用 $B$ 的结论解决 $A$，直观上说明 $A$ 语言对应的问题至少不会比 $B$ 对应的问题更难．因此归约是往更加复杂的语言上归约．

如果 $A \leqslant_T B$，并且 $B$ 可判定，即，既存在 $\text{M}_1^B$ 判定语言 $A$，亦存在 $\text{M}_2$ 判定语言 $B$，那么可以使用通用图灵机 $\text{U}$ 模拟 $\text{M}_1$，并在需要得知 $\text{M}_2$ 结果时，保存当前状态并模拟 $\text{M}_2$ 然后返回（比如可以利用很远处的纸带运行 $\text{M}_2$），于是 $A$ 可判定．其逆否命题为

**<font color=dodgerblue>定理 2.3</font>** 若 $A \leqslant_T B$ 且 $A$ 不可判定，则 $B$ 不可判定．

一个简单的例子是停机问题，该语言对应的图灵机一定停机，即 $\texttt{HALT} = \left\lbrace\langle{\text{M} }\rangle\right\vert\left.\forall{x},{\text{M} }(x)\downarrow\right\rbrace$．我们想证明它的不可判定性，根据性质 1.18，$A_{\mathbb{TM} }$ 不可判定，我们可以尝试用 $\texttt{HALT}$ 作为神谕，来让 $A_{\mathbb{TM} }$ 归约到它．

可以构造如下神谕图灵机 $\text{M}^\texttt{HALT}(\langle\text{T}\rangle, x)$，其中 $\text{T} \in \mathbb{TM}$，

- 构造一个图灵机 $\text{D}(\omega)$，对于任何 $\omega$，$\text{D}(\omega)$ 模拟运行 $\text{T}(x)$
- 询问神 $\langle{\text{D} }\rangle$ 是否属于 $\texttt{HALT}$
- 如果属于，则模拟运行 $\text{T}(x)$
- 如果不属于，则直接拒绝

于是 $\text{M}^\texttt{HALT}(\langle\text{T}\rangle, x) \cong A_{\mathbb{TM} }(\langle\text{T}\rangle, x)$，即有 $A_{\mathbb{TM} } \leqslant_T \texttt{HALT}$，又由于 $A_{\mathbb{TM} }$ 不可判定，所以

**<font color=dodgerblue>性质 2.4</font>** 语言 $\texttt{HALT}$ 不可判定．

### 2.3 映射归约

除了图灵归约，还有另一个看起来也很自然的归约方式．

**<font color=dodgerblue>定义 2.5</font>** 定义映射归约 (mapping reduction)，或称多一归约，如果存在可计算函数（定义 1.17）$f(x)$ 和语言 $A, B$ 满足 $x\in A \Leftrightarrow f(x)\in B$，则称 $A$ 语言可以映射归约到 $B$，记作 $A \leqslant_m B$．同时，这里的 $f$ 称为 $A$ 到 $B$ 的归约函数 (reduction function)．

如果 $B$ 语言可判定，设这个可以判定 $B$ 语言的图灵机是 $\text{T}$，那么构造一个图灵机 $\text{D}(x)$，首先将所有输入 $x$ 映射到 $f(x)$ 上，接着模拟运行可以判定 $B$ 的图灵机 $\text{T}(f(x))$，此时如果 $\text{T}(f(x))$ 接受说明 $f(x)\in B \Leftrightarrow x\in A$，拒绝亦然（两边同取否），这样就成功判定 $A$ 了．即如果 $B$ 语言可以判定，那么 $A$ 语言同样可以判定．因此，它有和图灵归约一样结论，即逆否命题

**<font color=dodgerblue>定理 2.6</font>** 若 $A \leqslant_m B$ 且 $A$ 不可判定，则 $B$ 不可判定．

这个归约有一个额外的性质，那就是可识别性同样继承了．刚才的证明中，如果 $B$ 仅仅是可识别，不一定可判定，即可能存在任意多的 $x \notin A$ 会使得 $T(f(x))\uparrow$ 即 $D(x)\uparrow$，也不影响 $\text{T}(f(x))$ 接受说明 $f(x)\in B \Leftrightarrow x\in A$ 的部分．因此，如果 $B$ 语言可以识别，那么 $A$ 语言同样可以识别．即逆否命题

**<font color=dodgerblue>定理 2.7</font>** 若 $A \leqslant_m B$ 且 $A$ 不可识别，则 $B$ 不可识别．

另一方面，如果 $B$ 仅仅是可识别补集（定义 1.11），完全相同的方式，可以证明 $A$ 语言也是可识别补集的．即

**<font color=dodgerblue>定理 2.8</font>** 若 $A \leqslant_m B$ 且 $A$ 不可识别补集，则 $B$ 不可识别补集．

即使我们不知道 $A$ 是否可识别，仅需要 $A$ 不可判定，再同时证明 $A \leqslant_m B$ 与 $A \leqslant_m \overline{B}$．由于 $A$ 不可识别或不可识别补集 $\Leftrightarrow$ $A$ 不可判定（推论 1.12），说明 $A$ 要么不可识别，同时将这个两个性质传递给了 $B$ 和 $\overline{B}$；要么不可识别补集，亦将这个两个性质传递给了 $B$ 和 $\overline{B}$，两种情况下，$B$ 和 $\overline{B}$ 都是不可识别的．即

**<font color=dodgerblue>推论 2.9</font>** 若 $A \leqslant_m B, A \leqslant_m \overline{B}$ 且 $A$ 不可判定，则 $B$ 不可识别且不可识别补集．

### 2.4 递归定理

**<font color=dodgerblue>定理 2.10</font>** 罗杰斯不动点定理 (Rogers's fixed-point theorem)表明，对于任意可计算函数 $f(x)$，存在图灵机 $\text{M}$ 使得 $\text{M}\cong_O\lbrack{f}(\langle{\text{M} }\rangle)\rbrack$．

证明：参考[此链接](https://en.wikipedia.org/wiki/Kleene%27s_recursion_theorem#Proof_of_the_fixed-point_theorem)．

**<font color=dodgerblue>定理 2.11</font>** 克林递归定理 (Kleene's recursion theorem)，亦称为第二递归定理 (the second recursion theorem)，表明，对于任意偏可计算（部分可计算）函数 $t(x, y)$，存在图灵机 $\text{M}$ 使得 $\text{M}(x)\cong_O{t}(\langle{\text{M} }\rangle, x)$．

证明：参考[此链接](https://en.wikipedia.org/wiki/Kleene%27s_recursion_theorem#Kleene's_second_recursion_theorem)

如果让 $t(x, y) \equiv x$，那么克林递归定理可以得到 $\text{M}(x)\cong_O{t}(\langle{\text{M} }\rangle, x) \equiv \langle{\text{M} }\rangle$，也即总存在一个图灵机能打印自己的序号．同样，这个序号可以改为代码，适用于计算机的 quine 程序的存在性证明．

**<font color=dodgerblue>定理 2.11</font>** 第一递归定理 (the first recursion theorem) 指出不动点定理中不仅存在函数，还存在一个“不动最小集”

### 2.A 本节符号表

| 符号 | 含义 |
| :---: | --- |
| $\text{M}^L$ | 神谕为 $L$ 的神谕图灵机 |
| $A \leqslant_T B$ | $A$ 可以图灵归约到 $B$ |
| $A \leqslant_m B$ | $A$ 可以映射归约到 $B$ |
