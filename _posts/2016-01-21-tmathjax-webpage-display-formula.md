---
title: MathJax:网页中显示精美公式
author: dyxu
layout: post
comments: true
permalink: /2016/01/21/mathjax-webpage-display-formula.html
categories:
  - 工具
tags:
  - 工具
  - MathJax
---

### MathJax是什么？

> MathJax是一个开源JavaScript库。它支持LaTeX、MathML、AsciiMath符号，可以运行于所有流行浏览器上。 
> 它的设计目标是利用最新的web技术，构建一个支持math的web平台。

MathJax是一款帮你在网页上快捷地显示漂亮公式的引擎，支持LaTex、MathML、AsciiMath等语法。本教程教大家如何使用
这个工具。

### 安装

MathJax的安装方式十分便捷，只要把官方提供的JavaScript的代码插入到网页中便可。（特别是用Jekyll在github上搭建博客的朋友，可以直接将以下这段代码复制到_includes/default.html的head标签内。）

	<script type="text/javascript"
		src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
	</script>

### Markdown中使用MathJax引擎

用MathJax引擎处理Markdown中的公式时，*.md文件依次经过了markdown引擎和MathJax引擎的解析，而\在前者中是转移字符，所以要用\\\代替在MathJax中的\，因此通过以下方式插入公式：

	$$公式$$         ==>行间公式(相当于latex的\begin{math}公式\end{begin})
	\\(公式\\)       ==>行内公式(相当于latex的\begin{displaymath}公式\end{displaymath})

例如正态分布`f(x) = \frac{1}{\sigma \sqrt{2\pi}}e^{-\frac{(x-\mu)^2}{2\sigma^2}}`MathJax解析后显示如下:

$$f(x) = \frac{1}{\sigma \sqrt{2\pi}}e^{-\frac{(x-\mu)^2}{2\sigma^2}}$$

而公式内的语法和LaTex相同（详情参见[常用数学符号的LaTeX表示方法](http://mohu.org/info/symbols/symbols.htm)),并且这些公式并不是图片，可以将鼠标移至图片上方进行**复制Tex代码**等操作。

### 公式引用

MathJax从2.0版本开始支持公式编号和引用。在公式内`\begin{equation}...\end{equation}`之间插入`\label{id}`给公式编号，然后在合适的地方`eqref{id}`即引用该公式。但是MathJax的默认设置中并没有加入这两个特性，需要在网页的head标签内加入以下代码：

	<script type="text/x-mathjax-config">
    		MathJax.Hub.Config({
        	TeX: {equationNumbers: {autoNumber: ["AMS"], useLabelIds: true}},
        	"HTML-CSS": {linebreaks: {automatic: true}},
        	SVG: {linebreaks: {automatic: true}}
 		});
	</script>

这样就可以为公式加编号和引用，例如：

	$$\begin{equation}J_\alpha(x) = \sum_{m=0}^\infty\frac{(-1)^m}{m!\Gamma(m+\alpha+1)} {\left({\frac{x}{2}}\right)}^{2m+\alpha}\label{J}\end{equation}$$

$$\begin{equation}J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \Gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha}\label{J}\end{equation}$$

在这里通过`\eqref{J}`引用上面的公式\eqref{J}。

以上。





