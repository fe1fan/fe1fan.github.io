+++
author = "Fe1Fan"
title = "Rust Gui 框架开发第一篇"
date = "2024-03-20"
description = ""
tags = [
    "rust",
]
categories = [
    "rust",
]
series = ["Eleven"]
+++

## 开篇

在很早之前使用 golang 做桌面应用程序的时候发现了一个特别有意思的库 [lorca](https://github.com/zserge/lorca)，趁着学习 rust 的机会，准备复刻一个 rust 的版本出来，所以才有了今天的 `Elevent` 系列。

![images](https://cdn.xka.io/elevent-01-01.png)

## 基本原理

在现有的跨平台的各类语言的 GUI 开源库中，比较有名的是 [electron](https://github.com/electron/electron)