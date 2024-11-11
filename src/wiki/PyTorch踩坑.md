---
title: Pytorch踩坑
aliases: 
tag: Pytorch
date: 2020-12-26 23:07:50
cover: http://pytorch.p2hp.com/assets/images/home-background.jpg
abbrlink: 6a6c6496
description: ed
---

# Pytorch踩坑

```python
torch.LongTensor(torch.zeros(1))
```

<!-- ![error1](../assets/116781023211267.png) -->

> 问题所在：`torch.Tensor`中不能传入`tensor`类型数据