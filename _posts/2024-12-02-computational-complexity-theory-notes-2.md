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

## 二、归约

### 2.1 神谕图灵机

**<font color=dodgerblue>定义 2.1</font>** 神谕图灵机 (oracle turing machine) 是这样一台图灵机：对于一个普通的图灵机 $\text{M}$ 和一个语言 $L$，如果 $\text{M}$ 在工作时可以直接询问“掌管 $L$ 的神”，以得知任何一个 $x$ 是否属于 $L$．则称这台图灵机 $\text{M}$ 是以 $L$ 为神谕的图灵机，记作 $\text{M}^L$．

如果一个 $L$ 是原本不可判定的语言，那么 $\text{M}^L$ 相比普通图灵机就能判定更多语言了．我们可以用 $\text{M}^L$ 引出对不可判定语言的另一种判断方式．

### 2.2 图灵归约

**<font color=dodgerblue>定义 2.2</font>** 对于语言 $A, B$，如果存在神谕图灵机 $\text{M}^B$ 判定语言 $A$，则称 $A$ 语言可以图灵归约 (turing reduction) 到 $B$ 语言，记作 $A \leqslant_T B$．

$\text{M}^B$ 判定语言 $A$，也即用 $B$ 的结论解决 $A$，直观上说明 $A$ 语言对应的问题至少不会比 $B$ 对应的问题更难．因此归约是往更加复杂的语言上归约．

如果 $A \leqslant_T B$，并且 $B$ 可判定，即，既存在 $\text{M}_1^B$ 判定语言 $A$，亦存在 $\text{M}_2$ 判定语言 $B$，那么可以使用通用图灵机 $\text{U}$ 模拟 $\text{M}_1$，并在需要得知 $\text{M}_2$ 结果时，保存当前状态并模拟 $\text{M}_2$ 然后返回（比如可以利用很远处的纸带运行 $\text{M}_2$），于是 $A$ 可判定．其逆否命题为

**<font color=dodgerblue>定理 2.3</font>** 若 $A \leqslant_T B$ 且 $A$ 不可判定，则 $B$ 不可判定．

一个简单的例子是停机问题，即 $\texttt{HALT} = \left\lbrace\langle{\text{M} }\rangle\right\vert\left.{\text{M} }\text{ halt in every }x\right\rbrace$．我们想证明它的不可判定性，根据性质 1.18，$A_{\mathbb{TM} }$ 不可判定，我们可以尝试用 $\texttt{HALT}$ 作为神谕，来让 $A_{\mathbb{TM} }$ 归约到它．

可以构造如下神谕图灵机 $\text{M}^\texttt{HALT}(\langle\text{T}\rangle, x)$，其中 $\text{T} \in \mathbb{TM}$，

- 构造一个图灵机 $\text{D}(\omega)$，对于任何 $\omega$，$\text{D}(\omega)$ 运行 $\text{T}(x)$
- 询问神 $\langle{\text{D} }\rangle$ 是否属于 $\texttt{HALT}$
- 如果属于，则模拟运行 $\text{T}(x)$
- 如果不属于，则直接拒绝

于是 $\text{M}^\texttt{HALT}(\langle\text{T}\rangle, x) \cong A_{\mathbb{TM} }(\langle\text{T}\rangle, x)$，即有 $A_{\mathbb{TM} } \leqslant_T \texttt{HALT}$，又由于 $A_{\mathbb{TM} }$ 不可判定，所以

**<font color=dodgerblue>性质 2.4</font>** 语言 $\texttt{HALT}$ 不可判定．
