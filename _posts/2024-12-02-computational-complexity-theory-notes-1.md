---
layout: post
title: "计算复杂性理论（一）"
date: 2024-12-02 14:32:15 +0800
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

计算机深刻地改变了当今世界．如今，在面对一个问题时，我们可能会认为，只需要有足够的计算能力，就可以解决这个问题了．但事实并非如此．在本节就能看到，即使“简单”如判定性问题，亦有无穷多个问题不能使用计算机（图灵机）来进行判定．

因此，在处理问题时，我们首先会判断它是否可以识别、计算（可识别性与可判定性），在可以计算的基础上，会关心计算它至少要花费多少时间与空间（时间复杂性类与空间复杂性类），是否有别的模型可以更进一步进行计算（非确定性图灵机、概率图灵机、交互图灵机等产生的复杂性类），以及，理论上是否存在一个可达的更快的方法（复杂性类的包含与等价关系，例如，$\mathbf{P} \overset{\text{?}}{=} \mathbf{NP}$）．

本节是计算复杂性理论 (Computational Complexity Theory)（亦称计算理论 (Theory of Computation)）的第一部分．在本节，我们将讲述什么是图灵机、语言，以及判定类的问题可以如何分类．

## 基础符号规定

1. 空字符串记为 $\epsilon$
2. 字符串 $\omega$ 的长度表示为 $\vert\omega\vert$
3. 若 $S$ 为符号集合，由 $S$ 中的符号串联构成的长度为 $n$ 的字符串的集合表示为 $S^n$，长度任意的字符串集合表示为 $S^\ast$（包含空字符串）  
   例：若 $S = \lbrace 0, 1\rbrace$，则 $S^2 = \lbrace 00, 01, 10, 11\rbrace$，$S^\ast = \lbrace \epsilon, 0, 1, 00, 01, 10, 11, 000, 001, \dots\rbrace$

## 一、图灵机与语言

### 1.1 图灵机

本文将采用三纸带图灵机 (turing machine) $\text{M}$，定义为拥有

- 三条两端均无限长的纸带 (tape)
  - 输入带（只读）input tape (read-only)
  - 工作带（读写）work tape (read/write)
  - 输出带（只写）output tape (write-only)
  - 每个带子上有一个指针，初始位置固定

- 有限符号 (symbol) 集 $\Gamma$
  - $\Gamma$ 包含空白符 $\square$ 和其他符号 $s_1, s_2, \dots, s_m$
  - 其他符号的集合表示为 $\Sigma$，即 $\Sigma = \left\lbrace s_1, s_2, \dots, s_m\right\rbrace$
  - 初始时，输入带上从初始位置向右写有输入 $\omega \in \Sigma^\ast$，输入带其他位置以及其他带子上都是空白符．输入为 $\omega$ 时可简记作 $\text{M}(\omega)$

- 有限状态 (state) 集 $Q$
  - $Q = \left\lbrace q_{\text{start} }, q_{\text{accept} }, q_{\text{reject} }, q_{1}, q_{2}, \dots, q_{n}\right\rbrace$
  - 其中，$q_{\text{start} }, q_{\text{accept} }, q_{\text{reject} }$ 分别表示开始、接受、拒绝状态，其他状态均为中间状态
  - 初始时为开始状态 $q_{\text{start} }$

- 迁移函数 (transition function) $\delta$  
  $$
  \delta: \left(Q\setminus\left\lbrace q_{\text{accept} }, q_{\text{reject} }\right\rbrace \right)\times\Gamma^2\to{Q}\times\Gamma^2\times\left\lbrace \text{L},\text{R},\text{S}\right\rbrace ^3 \\
  \delta\left(q_{now}, s_a, s_b\right) = \left(q_{next}, s_b', s_c', D_a, D_b, D_c\right)
  $$
  - 左侧 $s_a, s_b$ 为前两个带子当前指针读到的值，右侧 $s_b', s_c'$ 为后两个带子上写入的值
  - $\text{L},\text{R},\text{S}$ 表示在进行完这次读写后，三条带子的指针分别左移、右移、不动
  - 当当前状态在 $q_{now} \left(q_{now} \notin \left\lbrace q_{\text{accept} }, q_{\text{reject} }\right\rbrace \right)$ 且前两个读入值为 $s_a, s_b$ 时，即会进行一次 $\delta$ 操作．将状态改为 $q_{next}$，并将后两个带子上的值改为 $s_b', s_c'$，修改**后**，三条带子的指针分别进行 $D_a, D_b, D_c$ 方向的移动
  - 如果进入 $q_{\text{accept} }$ 或 $q_{\text{reject} }$ 状态，则运行终止，即停机 (halt)，并表示 $\text{M}$ 接受 (accept) 或拒绝 (reject) 输入 $\omega$
  - 如果图灵机在输入 $\omega$ 停机，记作 $\text{M}(\omega)\downarrow$；否则记作 $\text{M}(\omega)\uparrow$
  - 如果图灵机在停机时，输出带上写有内容 $\sigma$，则称图灵机输出 $\sigma$，记作 $\text{M}(\omega) = \sigma$

- 另外，
  - 迁移函数是一个偏函数 (partial function)，即对于某些 $\left(q_z, s_x, s_y\right)$ 可能没有定义．如果在运行中遇到没有定义的情况，机器将立刻停机，并拒绝该输入
  - 注意，图灵机可能不会停机

符号集需要至少包含两个符号（例如 $0, 1$），此外并无其他要求．我们可以编码将两个不同的符号集 $S_1, S_2$ 在 $\log_{\left\vert S_1\right\vert}(\left\vert S_2\right\vert)$ 的长度变化下由一个符号集映射到另一符号集，因此，不同的符号集在绝大多数情况下不影响结果．由于自然数和字符串均可数，可以一一对应，所以有时会直接用自然数表示输入．

我们可以让图灵机接收多个参数 $\omega_1, \omega_2, \dots, \omega_n$，多个参数之中可以用一个新字符隔开再重新编码为旧字符，或者让字符集一直拥有一个仅作为分割用的字符．此时，可简记作 $\text{M}\left(\omega_1, \omega_2, \dots, \omega_n\right)$．

在特定符号集下，所有图灵机构成的集合记为 $\mathbb{TM}$．

注1：一条纸带的定义亦可，但是防止后续对严格处理 $\mathbf{L}$ 的复杂性产生影响，本文规定为三条．  
注2：其他定义可能与之略有出入，但不影响使用．

### 1.2 图灵机编号

由于二进制编码 <sup>[link](https://en.wikipedia.org/wiki/Binary_code)</sup> 的存在，图灵机的迁移函数的所有输入输出情况可以写为一个列表，并编码为二进制，从而表示一个具体的图灵机．

接着，我们将图灵机的编码从小到大排序，并依次映射到自然数上．由于图灵机集合 $\mathbb{TM}$ 中的编码有无穷多个，因此 $\mathbb{TM}$ 的势至少是 $\aleph_0$（最小的无穷势）；又由于这些编码是 $\lbrace 0, 1\rbrace ^\ast$ 的子集，因此 $\mathbb{TM}$ 的势最大是 $\left\vert\lbrace 0, 1\rbrace ^\ast\right\vert=\aleph_0$．因此，它的势就是 $\aleph_0$，即 $\left\vert\mathbb{TM}\right\vert = \aleph_0$．因而，$\mathbb{TM}$ 和 $\mathbb{N}$ 等势，所以每个自然数 $n$ 都有一个对应的图灵机，每个图灵机 $\text{M}$ 也有一个对应的自然数，即

**<font color=dodgerblue>推论 1</font>** 自然数和图灵机一一对应．

本文用 $\lbrack{n}\rbrack$ 表示自然数 $n$ 对应的图灵机，$\lbrack{n}\rbrack\in\mathbb{TM}$；用 $\langle{\text{M} }\rangle$ 表示图灵机 $\text{M}$ 对应的自然数，$\langle{\text{M} }\rangle\in\mathbb{N}$．

### 1.3 语言

定义语言 (language) $L$ 如下，对于一个符号集 $\Sigma$ 所构成的字符串集合 $\Sigma^\ast$，语言 $L$ 是 $\Sigma^\ast$ 的一个子集，即 $L \subseteq \Sigma^\ast$．

语言 $L$ 常常用文字描述．例如，在符号集 $\Sigma = \left\lbrace a, b\right\rbrace$ 时，“回文字符串”可以指代一种语言 $L_{pal}$，$L_{pal} = \left\lbrace \epsilon, a, b, aa, bb, aaa, aba, bab, bbb, \dots\right\rbrace$．

对于一个符号集 $\Sigma$，所有语言 $L$ 构成的集合记为 $\mathbb{L}$，它是 $\Sigma^\ast$ 的幂集，即 $\mathbb{L} = 2^{\Sigma^\ast}$，它的势 $\left\vert\mathbb{L}\right\vert = \aleph_1$．

语言 $L$ 的补集对应的语言记为 $\overline{L}$，称为 $L$ 的补 (complement)．

语言的集合常常称为“类 (class)”，对于类 $\mathbf{R} = \left\lbrace  {L_1}, {L_2}, \dots\right\rbrace$，将 $\left\lbrace \overline{L_1}, \overline{L_2}, \dots\right\rbrace$ 记为 $\mathbf{co\text{-}R}$，称为 $\mathbf{R}$ 类的补类 (complement class)．

### 1.3 判定与识别

对于一台图灵机 $\text{M}\in\mathbb{TM}$，符号集 $\Sigma$，若对任何 $\omega\in\Sigma^\ast$ 作为输入，$\text{M}$ 一定停机，即一定接受 $\omega$ 或者拒绝 $\omega$，则将所有接受的 $\omega$ 组成的集合记为语言 $L$，称 $\text{M}$ **判定 (decide)** $L$．

如果图灵机 $\text{M}$ 对任何 $\omega\in\Sigma^\ast$ 作为输入，$\text{M}$ 接受 $\omega$ 当且仅当 $\omega\in{L}$，则称 $\text{M}$ **识别 (recognize)** $L$．和上文判定的区别是，识别 $L$ 时 $\text{M}$ 不一定停机，即在 $\omega\notin{L}$ 时，$\text{M}$ 可以拒绝，也可以死循环．

$\text{M}$ 输入为多个参数时可以记作 $\left(\omega_1, \omega_2, \dots, \omega_n\right) \in L$．

每个图灵机一定识别一种语言（只考虑接受，不停机和拒绝不用处理；哪怕全部拒绝或不停机，也识别了语言 $L=\lbrace \epsilon\rbrace$），但只有一定停机的的图灵机才能识别并判定一种语言．另外，在判定、识别语言时，无需考虑输出，仅需进入 $q_{\text{accept} }$、$q_{\text{reject} }$ 状态即可．

对于一个语言 $L$，如果存在图灵机判定它，则称 $L$ 是**可判定 (decidable)** 或**可计算 (computable)** 的．

对于一个语言 $L$，如果存在图灵机识别它，则称 $L$ 是**可识别 (recognizable)** 的．
  
**<font color=dodgerblue>推论 2</font>** 由定义可知，

- $L$、$\overline{L}$ 可识别 $\Leftrightarrow$ $L$ 可判定 $\Leftrightarrow$ $\overline{L}$ 可判定
- $L$、$\overline{L}$ 至少有一个不可识别 $\Leftrightarrow$ $L$ 不可判定

**<font color=dodgerblue>推论 3</font>** 由于 $\left\vert\mathbb{L}\right\vert = \aleph_1 \gg \aleph_0 = \left\vert\mathbb{TM}\right\vert$，而一种图灵机最多只能识别、判定一种语言，所以有无穷多的语言是不可识别且不可判定的．

### 1.4 图灵机等价性

对于一个**判定性**问题，我们关心真假值．如果两个图灵机 $\text{M}_1, \text{M}_2\in\mathbb{TM}$ 对任何 $\omega$ 作为输入，都有

$$
\left\lbrace
\begin{aligned}
& \text{M}_1 \text{ accept } \omega \text{ iff } \text{M}_2 \text{ accept } \omega \\
& \text{M}_1 \text{ reject } \omega \text{ iff } \text{M}_2 \text{ reject } \omega \\
& \text{M}_1(\omega)\uparrow \text{iff } \text{M}_2(\omega)\uparrow \\
\end{aligned}
\right.
$$

则称 $\text{M}_1, \text{M}_2$ 等价，记作 $\text{M}_1 \cong \text{M}_2$，在一些要展示参数位置的情况，可以写成 $\text{M}_1(\omega) \cong \text{M}_2(\omega)$．

同样，如果定义在 $\Sigma^\ast \to \lbrace 0, 1 \rbrace$（或对应的 $\mathbb{N} \to \lbrace 0, 1 \rbrace$）上的函数 $f(x)$ 满足

$$
\left\lbrace
\begin{aligned}
& f(x) = 1 \text{ iff } \text{M} \text{ accept } x \\
& f(x) = 0 \text{ iff } \text{M} \text{ reject } x \\
& f(x) \text{ not defined iff } \text{M}(x)\uparrow \\
\end{aligned}
\right.
$$

记作 $f \cong \text{M}$ 或 $f(x) \cong \text{M}(x)$．如果函数 $f$ 在所有字符串（自然数）上均有定义，则称为全函数，否则叫做偏函数．

注3：如果我们在意输出值（并非判定性问题），可能会需要这样的关系

$$
\left\lbrace
\begin{aligned}
& f(x) = \text{M}(x) \text{ iff } \text{M}(x)\downarrow \\
& f(x) \text{ not defined iff } \text{M}(x)\uparrow \\
\end{aligned}
\right.
$$

图灵机之间的关系同理．此时，该问题与语言无关（可能是一个求解问题），且不关心拒绝接受．这种问题和等价性出现时，会有非常明显的提示指明问题具有输出，不过现在暂且不必关心．

**<font color=dodgerblue>定理 4</font>** 存在通用图灵机 $\text{U}$，对任意 $\langle{\text{M} }\rangle$ 和 $\omega$ 输入，使得 $\text{U}(\langle{\text{M} }\rangle, \omega) \cong \text{M}(\omega)$．参考[链接](https://en.wikipedia.org/wiki/UTM_theorem)．

### 1.A 本节符号表

| 符号 | 含义 |
| :---: | --- |
| $\mathbb{TM}$ | 图灵机集合 |
| $\text{M}$ | 图灵机 |
| $\text{M} \text{ accept } \omega$ | 图灵机接受 $\omega$ |
| $\text{M} \text{ reject } \omega$ | 图灵机拒绝 $\omega$ |
| $\text{M}(\omega)\downarrow$ | 图灵机 $\text{M}$ 在输入 $\omega$ 时停机 |
| $\text{M}(\omega)\uparrow$ | 图灵机 $\text{M}$ 在输入 $\omega$ 时不停机 |
| $\text{M}(\omega) = \sigma$ | 图灵机 $\text{M}$ 在停机时输出 $\sigma$ |
| $\text{M}\left(\omega_1, \omega_2, \dots, \omega_n\right)$ | 图灵机 $\text{M}$ 以 $\omega_1, \omega_2, \dots, \omega_n$ 作为输入 |
| $\lbrack{n}\rbrack$ | 自然数 $n$ 对应的图灵机 |
| $\langle{\text{M} }\rangle$ | 图灵机 $\text{M}$ 对应的自然数 |
| $\mathbb{L}$ | 语言集合 |
| $L$ | 语言 |
| $\overline{L}$ | 语言的补 |
| $\mathbf{R}$ | 语言类 |
| $\mathbf{oc\text{-}R}$ | 语言类 $\mathbf{R}$ 的补类 |
| $\text{M}_1(\omega) \cong \text{M}_2(\omega)$ | 图灵机之间等价 |
| $f(x) \cong \text{M}(x)$ | 函数和图灵机等价 |

### 1.B 本节例子

1，一个可识别可判定的例子：回文语言 $L_{pal}$

证明：图灵机首先在工作带照抄一遍输入，然后将输入带和工作带的指针分别移动到头和尾，然后分别向右向左运动并比较两个字符是否相同．如果遇到不相同的则拒绝，如果遇到空白（扫描完毕）则接受．

2，一个可识别不可判定的例子：$A_{\mathbb{TM} }$

$A_{\mathbb{TM} }$ 判断图灵机 $\text{M}$ 是否接受一个输入 $\omega$，$(\langle{\text{M} }\rangle,\omega) \in A_{\mathbb{TM} }$ 当且仅当 $\text{M}$ 图灵机接受 $\omega$

证明：对角线方法，参考[链接](https://courses.cs.washington.edu/courses/cse322/04au/Lect10.pdf)．此证明方法十分重要．

**<font color=dodgerblue>性质 5</font>** $A_{\mathbb{TM} }$ 可识别，但不可判定

3，一个不可识别不可判定的例子：$\overline{A_{\mathbb{TM} }}$

证明：由于 $A_{\mathbb{TM} }$ 可识别不可判定，根据前文定义中的直接推论，$L$、$\overline{L}$ 至少有一个不可识别 $\Leftrightarrow$ $L$ 不可判定，因此 $\overline{A_{\mathbb{TM} }}$ 不可识别，亦不可判定．
