---
layout: post
title:  "配置VSCode到Github"
date:   2026-08-12
categories: 日常 学习 GitHub VSCode
---

换台电脑尝试使用更新, 发现github pages设置有问题, 原来的页面使用master分支, VSCode会提交到新建立的分至main. 在Github **Settings**-> **Pages**

**Source**一栏选择**Deploy from a branch**, 分支选择新的**main**, 目录选择/ (root). 正常设置后从VSCode提交修改就会触发重新构建页面.

