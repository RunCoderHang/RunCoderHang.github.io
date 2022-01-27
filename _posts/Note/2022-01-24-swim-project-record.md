---
title:  泳池项目实验记录（十）
tags: 
date: 2022-01-24 23:30:00
categories: 学习总结
---



本周主要的工作：根据上次甲方会议内容，要将泳池中的安全人数进行稳定处理；添加图像检测救生员是否在岗的功能；我上周所提出的要将跟踪算法进行替换。

- 泳池安全人数稳定

目前我们计算出来的人数是图像中所有检测到的人数（包括岸上的人以及泳池内的人）。在之前的测试中，我们就发现：当人在游泳的时候，难免会做出下潜的动作。当人下潜后，摄像头就几乎检测不到这个正在游泳的人。因此，我们在实时检测中的检测人数会不停的在上下跳动。

最后拟定的方案就是：添加极大值滤波。也就是在读取视频帧时，每次记录一段时间的视频帧，并且记录每帧中的检测人数，最后我们去对这所有检测帧的人数去一个最大值并返回。这样的 trick，能够有效较少数字跳动的问题。

缺点：因为是每次要记录一段的视频帧数据，因此也会耗费一些内存。并且，如果这个视频帧长度没有调整好，那么就会导致我们返回的最终人数是一直以最先前帧中的最大人数，也就是人数变换的延迟时间大大增加。

记录一段代码，可以作为 baseline 使用：

```python
import time
import numpy as np


class PredictionFilter:
    def __init__(self, interval_time=1):
        self.state_list = []
        self.interval_time = interval_time

    def filter(self, state):
        self.state_list.append((state, time.time()))
        now = time.time()
        self.state_list = [state for state in self.state_list if now - state[1] < self.interval_time]
        res_list = []
        for state in self.state_list:
            res_list.append(state[0])
        return np.max(res_list)
```

- 监控救生员是否在岗

该功能其实是对指定的相机进行划区域检测。首先使用 json 文件中写入该相机的指定区域的坐标，然后代码中读取每个点的坐标。在遍历每个目标的坐标时，就可以判断是否有目标在该区域内。如果有一个在的话，并且会持续很长时间，则认为该救生员正在岗值班。

目前只有三个相机能看见救生员的值班区域，因此需要对这个三个相机进行指定区域的识别。每个相机都需要初始化一个类去记录是否在岗信息。


- 跟踪算法替换

将 DeepSORT 替换成 SORT ，离线已经完成，之后需要部署到线上去使用。




