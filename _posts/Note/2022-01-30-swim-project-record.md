---
title:  泳池项目实验记录（十一）
tags: 
date: 2022-01-30 23:30:00
categories: 学习总结
---


- 本周主要工作：处理数据、检查数据、用新数据去训练新的模型
- To-do：对人头人体匹配算法中的 IOU 进行优化

1. 训练新模型

首先说一下本周工作内容：写脚本对新一批数据中的 json 文件和图片进行处理。之前一直去处理新一批数据，一是因为没有时间，二是因为本次新一批数据格式又有了新的变化，又需要重新写一个脚本去将 json 文件分割。为什么说“又”，因为前几批数据中有几次也是不一样的。在这里稍微吐槽一下。

主要工作就是将 json 文件分离、 json 中 bounding box 坐标点作修改（负值改为 0 ）、图片从原文件夹中分离等等。由于要训练新的模型，所以对于前几批数据的标注还要进行修改。然后为了测试修改后的 bounding box 是否准确，又写了一个根据 json 和 xml 中的标注进行可视化的脚本。一系列工作下来也没发现有什么大的问题。

最后，就是训练新模型（检测）。之前的检测模型是对人体、人头水上、人头水下进行检测。为了完成甲方提出的：剔除上的人员数量，仅对泳池内人员进行数量计算，那么就需要对人体进行水上和水下的分类。因此本次训练的新模型就是对人体水上、人体水下、人头水上、人头水下进行检测分类。

2. 匹配算法 IOU 优化

最后就是要对匹配算法进行优化的想法，在这里记录一下。

之前的 IOU 计算是人头框在保证不超出人体框的范围内进行 IOU 计算，但是缺点就是如果如果人头框超出了人体框的范围，那么我们的人头人体匹配算法就会失败。因此，本次优化的想法就是去改进这一缺点。


![761d593ccad4729d5927f15abfef3aa3.png](https://runcoderhang.github.io/thumbnails/7a2132d90e834dd587d3f47a0040a7df.png)
已匹配（人头框未超出人体框）

![35609af00ae8b386abe6c2a4ce456f87.png](https://runcoderhang.github.io/thumbnails/1dc83fe343994276942fd9b000e0e1f5.png)
未匹配（人头框超出人体框）

改进想法：当人头框超出人体框的范围时，我们计算人头框超出人体框的面积占人头框总面积的比例。并设置一个阈值，限制这个面积比的大小，我们就认为只要在这个阈值内，并且认为只要不超过这个阈值内的人头框和人体框可以作为一个人匹配。

