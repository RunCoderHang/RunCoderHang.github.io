---
title:  泳池项目实验记录（七）
tags: 
date: 2022-01-03 23:30:00
categories: 学习总结
---

本周主要工作是针对bug进行测试以及分析。我们一开始分析是检测模块出现了问题，因为是在跟踪结果中的相同目标上说到底出现了两个跟踪框，然而跟踪框说到底是从检测框中拿到的结果，也就是说我们的跟踪框是检测框的子集。

那么，既然跟踪框在相同的目标上出现了两种框，是不是就是检测器中就将这个目标检测成了两个目标，因此才得到两个检测框，然后传给跟踪器去跟踪。

随后，我们使用了两种方案去分析该问题：

- 方案一：使用检测器输出检测结果后进行匹配，随后查看仅检测器输出的结果在线上的相机中是否出现了问题。
- 方案二：使用 检测 $\to$ 匹配 $\to$ 跟踪 的流程来查看结果是否有问题，也就是去排查是否是因为跟踪模块出现了bug。

最终，使用两个方案查看输出结果后，发现是在跟踪模块中出现了问题。接下来的工作，就去查看该bug是在跟踪模块的哪个节点处才会发生。以及是否是因为线上代码或输入流与线下有所不同才会产生次bug。

接下来的工作就是：

- 设计一个 IOU 代码，当发生这种两个框同时出现的时候，它们的 IOU 会重叠很严重，这时候就输出该节点信息进行分析。
- 将线上代码进行本地测试，查看是否是代码原因还是视频输入的原因。

