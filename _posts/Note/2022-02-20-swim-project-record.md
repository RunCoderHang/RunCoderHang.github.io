---
title:  泳池项目实验记录（十四）
tags: 
date: 2022-02-20 23:30:00
categories: 学习总结
---


本周工作：

将项目代码进行不同模块的划分。因为不管是线上代码还是线下代码的编写，都没有注意代码的复用性以及维护问题。这导致在调参的过程中，需要动用很多的地方才能进行调整。因此，我将代码的每个功能模块又重构了一遍，并在新的项目地址上进行了修改。


```
|-- swim
    |-- Calibration
    |-- Demos
    |   |-- mot_offline (已弃)
    |       |-- Tracker_demo.py
    |       |-- Tracker_demo_new.py
    |   |-- mot_online
    |       |-- track_sdk.py
    |       |-- track_sdk_4cls.py
    |       |-- track_test.py
    |       |-- track_test_4cls.py
    |   |-- mot_v1 (已弃)
    |-- SwimMOT
    |   |-- byte_track
    |   |-- sort
    |   |-- deep_sort_pytorch
    |   |-- yolov5
    |   |-- utils
    |       |-- general.py  
    |       |-- get_json.py
    |       |-- iou_match.py
    |       |-- lifeguard_detect.py
    |       |-- predict_drown.py
    |       |-- predict_detect_number.py
    |       |-- predict_lifeguard.py
    |   |-- old_demo (已弃)
    |-- hang_detector.py
    |-- hang_byte_tracker.py
    |-- hang_tracker_sort.py
    |-- hang_tracker.py
```

`mot_online` 中存放的是服务器在线运行程序和在线测试的程序

- `track_sdk.py`
- `track_sdk_4cls.py`
- `track_test.py`
- `track_test_4cls.py`


`utils` 中的一些工具：

- `general.py` ：存放一些可视化的工具，例如画框、转换坐标框
- `get_json.py` ：将跟踪结果
- `iou_match.py` ：存放人头和人体的匹配算法，以及框之间的 IOU 计算
- `lifeguard_detect.py` ：存放救生员是否在岗功能
- `predict_drown.py` ：游泳人员溺水判断功能
- `predict_detect_number.py` ：极大值滤波将检测到的游泳人数进行稳定
- `predict_lifeguard.py` ：采用滤波方式稳定救生员在岗检测情况


还有定义了检测器的接口以及其他各种跟踪算法的接口：

- `hang_detector.py`
- `hang_byte_tracker.py`
- `hang_tracker_sort.py`
- `hang_tracker.py`



