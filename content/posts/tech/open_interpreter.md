+++
title = 'open_interpreter 学习' 
date = 2023-09-23T21:46:05+08:00
lastmod = 2023-09-23T21:46:05+08:00
author = ["geekhuashan"]
draft = true
categories = ["life", "tech"]
tags = ["life", "tech"]
description = ""
weight = 2
hidemeta = false
+++

## 使用open_interpreter 的原因

在Youtube 王树义老师，以及小众软件的介绍下，我了解到了open interpreter

从open interpreter的介绍中，我发现了一些有趣的东西，比如说，这个是一个对标openai官方的advanced数据分析工具，它有很多的优势。

对比openai 的官方服务：

- No internet access. 没有网络访问
- Limited set of pre-installed packages. 有限的预安装包
- 100 MB maximum upload, 120.0 second runtime limit. 100MB的最大上传，120秒的运行时间限制
- State is cleared (along with any generated files or links) when the environment dies. 当环境死亡时，状态被清除（以及任何生成的文件或链接）

这些限制，让我想到了，我可以通过这个工具，来完成一些有趣的事情，在本地分析一个大的数据集，安装很多官方不允许的包，然后通过这个工具来完成分析，而且我在公司由于层层的审查，变得越来越不自由，公司由于是用的微软的云服务，我相信这个azure的api能够在公司通行无阻的应用，这样我就可以在公司分析一些有趣的事情，而且可以通过 command line 来多完成一些事情。

## 本地安装


## 有趣应用实例

### 1. 通过open interpreter 来完成一个数据分析的流程