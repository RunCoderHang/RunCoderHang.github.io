---
title:  泳池项目实验记录（五）
tags: 
date: 2021-12-12 23:30:00
categories: 学习总结
---


本周主要工作：将多相机联合跟踪功能放在服务器中，并成功运行。不过期间还是出现了一些问题：

首先是犯了一个低级的错误：在多相机实现跟踪过程中，需要将每个相机的图形帧放置一个列表中，然后传入检测器中进行检测。检测的结果也是一个列表，放置的是每个相机图像帧的目标检测结果。随后，我将所有的检测结果放置在一个跟踪器中，结果我所得到的跟踪结果是为空的。

因为，我们的跟踪器中需要传入的是具有相互关联的检测结果。例如，我们要传入的是一个视频，视频中的前后每一帧里的目标都是有着相互关联的信息，如果我们传入的是没有相互关联的检测结果，那么所得到的跟踪结果则是为空的。

最后，我初始化六个跟踪器，每个跟踪器用于每个摄像头。这样，每个跟踪器所得到的检测结果必然是前后帧信息是相互关联的。

代码如下：

```python
def get_track_results(detection_results, names, tracker_list, frame_list):
    track_results = []
    img0_list = []
    for frame in frame_list:
        frame = cv2.resize(frame, (int(frame.shape[1] * 4), int(frame.shape[0] * 4)))
        img0_list.append(frame)
    for i in range(len(detection_results)):
        track = tracker_list[i].track(detection_results[i], names, img0_list[i])
        track_results.append(track)
    return track_results


if __name__ == '__main__':
    cfg = get_config()
    cfg.merge_from_file('config/hang_detector.yaml')
    cfg.merge_from_file('config/hang_tracker.yaml')
    
    # init detector
    detect = hang_detector.Detect(cfg)
    detect.load_model()
    
    # init six trackers for every camera
    tracker_1 = hang_tracker.Tracker(cfg)
    tracker_2 = hang_tracker.Tracker(cfg)
    tracker_3 = hang_tracker.Tracker(cfg)
    tracker_4 = hang_tracker.Tracker(cfg)
    tracker_5 = hang_tracker.Tracker(cfg)
    tracker_6 = hang_tracker.Tracker(cfg)
    tracker_list = [tracker_1, tracker_2, tracker_3, tracker_4, tracker_5, tracker_6]

    # 对于每一帧都要预测溺水,因此对于每一个相机的跟踪结果都要先初始化
    predictor_1 = DrownPredict()
    predictor_2 = DrownPredict()
    predictor_3 = DrownPredict()
    predictor_4 = DrownPredict()
    predictor_5 = DrownPredict()
    predictor_6 = DrownPredict()
    predictor_list = [predictor_1, predictor_2, predictor_3, predictor_4, predictor_5, predictor_6]

    camera_ids = ["192.168.1.2", "192.168.1.3", "192.168.1.4", "192.168.1.5", "192.168.1.6", "192.168.1.7"]

    # init http post
    https_post = HttpsPost()

    # init drown prediction
    drown_predict = DrownPredict()

    # init number
    number_detect = NumberDetect(16)

    # init filter
    pred_filter = PredictionFilter()
    
    while 1:
        frame_list1 = cam.camera_reads()
        frame_list2 = cam.camera_reads()
        frame_list = choosebest(frame_list1, frame_list2)
        detect_time = frame_list.pop(6)

        detection_results, names = detect.detect(frame_list)

        track_results = get_track_results(detection_results, names, tracker_list, frame_list)

        json_data = to_json(track_results, predictor_list, camera_ids, frame_list)

        https_post.post(json_data)
```

