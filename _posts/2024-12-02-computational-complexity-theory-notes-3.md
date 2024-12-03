---
layout: post
title: "计算复杂性理论（三）"
date: 2024-12-03 21:24:00 +0800
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

## 三、非确定图灵机与复杂性类
