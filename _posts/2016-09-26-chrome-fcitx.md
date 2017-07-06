---
layout: post
title: Chrome漏字母解决方案
author: dyxu
keywords: 
description:
categories:
  - 解决方案
tags: ["程序员"]

---
{% include JB/setup %}

## 问题描述

Linux版本Chrome浏览器在打字速度快时会漏字母。例如：
我要打“今天天气真好” （jintiantianqizhenhao)
打得快时会变成：
“tq几年踢啊你真好”（jin(t)iantian(q)izhenhao)
两个字母直接到屏幕上了。

## 解决方法

应该是安装的fcitx版本有问题，安装fcitx-gtk2就可以了。
