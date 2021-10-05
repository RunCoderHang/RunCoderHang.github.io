---
title:  DeepSORT之代码解析（二）
tags: 
- DeepSORT
date: 2021-09-26 23:30:00
categories: 学习总结
---

上一次，是讲解DeepSORT代码部分的各个模块，本次主要是梳理DeepSORT代码运行的流程。

本人使用的 Yolov5 + DeepSORT 的整体代码，所作出的分析也是基于此代码。代码地址在 [https://github.com/mikel-brostrom/Yolov5_DeepSort_Pytorch](https://github.com/mikel-brostrom/Yolov5_DeepSort_Pytorch) 。

## 流程分析

流程部分主要按照以下流程图来走一遍：[图片来自博客]

![f406d57321dd96a27b1770b4305f326c.png](https://runcoderhang.github.io/thumbnails/0c560cd9fecf49e6acd40eb6cdf555c7.png)

### 1. track.py

当 Yolov5 检测模型和跟踪模型都准备好以后，开始运行代码测试时，我们就从该运行代码 `track.py` 开始

```python
def detect(opt):
    out, source, yolo_weights, deep_sort_weights, show_vid, save_vid, save_txt, imgsz, evaluate = \
        opt.output, opt.source, opt.yolo_weights, opt.deep_sort_weights, opt.show_vid, opt.save_vid, \
            opt.save_txt, opt.img_size, opt.evaluate
    webcam = source == '0' or source.startswith(
        'rtsp') or source.startswith('http') or source.endswith('.txt')

    # initialize deepsort
    cfg = get_config()
    cfg.merge_from_file(opt.config_deepsort)
    attempt_download(deep_sort_weights, repo='mikel-brostrom/Yolov5_DeepSort_Pytorch')
    # deepsort 运行
    deepsort = DeepSort(cfg.DEEPSORT.REID_CKPT,
                        max_dist=cfg.DEEPSORT.MAX_DIST, min_confidence=cfg.DEEPSORT.MIN_CONFIDENCE,
                        max_iou_distance=cfg.DEEPSORT.MAX_IOU_DISTANCE,
                        max_age=cfg.DEEPSORT.MAX_AGE, n_init=cfg.DEEPSORT.N_INIT, nn_budget=cfg.DEEPSORT.NN_BUDGET,
                        use_cuda=True)

```

初始化DeepSORT对象，更新部分接收目标检测得到的框的位置，置信度和图片：

```python
# deepsort 跟踪计算
# pass detections to deepsort
outputs = deepsort.update(xywhs.cpu(), confs.cpu(), clss.cpu(), im0)
```

### 2. 根据 deepsort 类的 update 函数看

```python
class DeepSort(object):
    def __init__(self, model_path, max_dist=0.2, min_confidence=0.3, max_iou_distance=0.7, max_age=70, n_init=3, nn_budget=100, use_cuda=True):

        # yolo中检测结果置信度阈值，筛选置信度小于0.3的detection
        self.min_confidence = min_confidence

        # 用于提取图片的embedding,返回的是一个batch图片对应的特征
        self.extractor = Extractor(model_path, use_cuda=use_cuda)

        # 用在级联匹配的地方，如果大于改阈值，就直接忽略
        max_cosine_distance = max_dist
        # 第一个参数可选'cosine' or 'euclidean'
        metric = NearestNeighborDistanceMetric(
            "cosine", max_cosine_distance, nn_budget)
        self.tracker = Tracker(
            metric, max_iou_distance=max_iou_distance, max_age=max_age, n_init=n_init)

    # 匹配、跟踪 都在 update 中
    def update(self, bbox_xywh, confidences, classes, ori_img):
        self.height, self.width = ori_img.shape[:2]
        # generate detections
        # 从原图中抠取bbox对应图片并计算得到相应的特征
        features = self._get_features(bbox_xywh, ori_img)
        bbox_tlwh = self._xywh_to_tlwh(bbox_xywh)

        # 筛选掉小于 min_confidence 的目标，并构造一个 Detection 对象构成的列表
        # Detection是一个存储图中一个bbox结果
        # 需要：1. bbox(tlwh形式) 2. 对应置信度 3. 对应embedding
        detections = [Detection(bbox_tlwh[i], conf, features[i]) for i, conf in enumerate(
            confidences) if conf > self.min_confidence]


        # run on non-maximum supression
        boxes = np.array([d.tlwh for d in detections])
        scores = np.array([d.confidence for d in detections])

        # 使用非极大抑制
        # 默认nms_thres=1的时候开启也没有用，实际上并没有进行非极大抑制
        # indices = non_max_suppression(boxes, self.nms_max_overlap, scores)
        # detections = [detections[i] for i in indices]

        # update tracker
        # tracker给出一个预测结果，然后将detection传入，进行卡尔曼滤波操作
        self.tracker.predict() # 将跟踪状态分布向前传播一步
        self.tracker.update(detections, classes) # 执行测量更新和跟踪管理

        # output bbox identities
        # 存储结果以及可视化
        outputs = []
        for track in self.tracker.tracks:
            if not track.is_confirmed() or track.time_since_update > 1:
                continue
            box = track.to_tlwh()
            x1, y1, x2, y2 = self._tlwh_to_xyxy(box)
            track_id = track.track_id
            class_id = track.class_id
            outputs.append(np.array([x1, y1, x2, y2, track_id, class_id], dtype=np.int))
        if len(outputs) > 0:
            outputs = np.stack(outputs, axis=0)
        return outputs
```

从这里开始对照以上流程图会更加清晰。在 deepsort 初始化的过程中有一个核心 metric ， `NearestNeighborDistanceMetric` 类会在匹配和特征集更新的时候用到。

梳理 DeepSORT 的 update 流程：

- 根据传入的参数（bbox_xywh, conf, img）使用ReID模型提取对应bbox的表观特征。
- 构建 detections 的列表，列表中的内容就是 `Detection` 类,在此处限制了bbox的最小置信度。
- 使用非极大抑制算法，由于默认nms_thres=1，实际上并没有用。
- Tracker 类进行一次预测，然后将 detections 传入，进行更新。
- 最后将 Tracker 中保存的轨迹中状态属于确认态的轨迹返回。

以上核心在 Tracker 的 `predict` 和 `update` 函数，接着梳理。

### 3. Tracker 的 predict 函数

Tracker 是一个多目标跟踪器，保存了很多个 track 轨迹，负责调用卡尔曼滤波来预测 track 的新状态+进行匹配工作+初始化第一帧。 Tracker 调用 update 或 predict 的时候，其中的每个 track 也会各自调用自己的 update 或 predict 。

```python
class Tracker:
    def __init__(self, metric, max_iou_distance=0.7, max_age=70, n_init=3):
        self.metric = metric
        self.max_iou_distance = max_iou_distance  # 最大iou，iou匹配的时候使用
        self.max_age = max_age # 直接指定级联匹配的cascade_depth参数
        self.n_init = n_init # n_init代表需要n_init次数的update才会将track状态设置为confirmed

        self.kf = kalman_filter.KalmanFilter() # 实例化卡尔曼滤波器
        self.tracks = [] # 保存一个轨迹列表，用于保存一系列轨迹
        self._next_id = 1 # 下一个分配的轨迹id

    def predict(self):
        """Propagate track state distributions one time step forward.
        将跟踪状态分布向前传播一步

        This function should be called once every time step, before `update`.
        """
        # 遍历每个track都进行一次预测
        for track in self.tracks:
            track.predict(self.kf)
```

predict 主要是对轨迹列表中所有的轨迹使用卡尔曼滤波算法进行状态的预测。

### Tracker 的 update 函数

Tracker的更新属于最核心的部分。

```python
def update(self, detections, classes):
    """Perform measurement update and track management.
    执行测量更新和轨迹管理

    Parameters
    ----------
    detections : List[deep_sort.detection.Detection]
        A list of detections at the current time step.
    """
    # Run matching cascade.
    matches, unmatched_tracks, unmatched_detections = \
        self._match(detections)

    # Update track set.

    # 1. 针对匹配上的结果
    for track_idx, detection_idx in matches:
        # 更新 tracks 中相应的 detection
        self.tracks[track_idx].update(
            self.kf, detections[detection_idx], classes[detection_idx])

    # 2. 针对未匹配的 track , 调用 mark_missed 进行标记
    # track 失配时，若 Tantative 则删除；若 update 时间很久也删除
    # max age是一个存活期限，默认为70帧
    for track_idx in unmatched_tracks:
        self.tracks[track_idx].mark_missed()

    # 3. 针对未匹配的 detection ，detection失配，进行初始化
    for detection_idx in unmatched_detections:
        self._initiate_track(detections[detection_idx], classes[detection_idx].item())
        
    # 得到最新的tracks列表，保存的是标记为confirmed和Tentative的track
    self.tracks = [t for t in self.tracks if not t.is_deleted()]
    
    # Update distance metric.
    active_targets = [t.track_id for t in self.tracks if t.is_confirmed()]
    # 获取所有confirmed状态的track id
    features, targets = [], []
    for track in self.tracks:
        if not track.is_confirmed():
            continue
        features += track.features
        # 获取每个feature对应的track id
        targets += [track.track_id for _ in track.features]
        track.features = []
        
    # 距离度量中的 特征集更新
    self.metric.partial_fit(
        np.asarray(features), np.asarray(targets), active_targets)
```

这部分注释已经很详细了，主要是一些后处理代码，需要关注的是对匹配上的，未匹配的 Detection ，未匹配的 Trac k三者进行的处理以及最后进行特征集更新部分，可以对照流程图梳理。

Tracker 的 update 函数的核心函数是 `match` 函数，描述如何进行匹配的流程：

```python
def _match(self, detections):
    # 主要功能是进行匹配，找到匹配的，未匹配的部分
    def gated_metric(tracks, dets, track_indices, detection_indices):
        features = np.array([dets[i].feature for i in detection_indices])
        targets = np.array([tracks[i].track_id for i in track_indices])

        # 1. 通过最近邻（余弦距离）计算出 cost_matrix （代价矩阵）
        cost_matrix = self.metric.distance(features, targets)

        # 2. 计算指派后的成本矩阵（代价矩阵）
        cost_matrix = linear_assignment.gate_cost_matrix(
            self.kf, cost_matrix, tracks, dets, track_indices,
            detection_indices)

        return cost_matrix

    # Split track set into confirmed and unconfirmed tracks.
    # 区分开 confirmed tracks 和 unconfirmed tracks 不同轨迹的状态
    confirmed_tracks = [
        i for i, t in enumerate(self.tracks) if t.is_confirmed()]
    unconfirmed_tracks = [
        i for i, t in enumerate(self.tracks) if not t.is_confirmed()]

    # Associate confirmed tracks using appearance features.
    # 进行级联匹配，得到匹配的track、不匹配的track、不匹配的detection
    '''
    !!!!!!!!!!!
    级联匹配
    !!!!!!!!!!!
    '''
    # gated_metric->cosine distance
    # 仅仅对确定态的轨迹进行级联匹配
    matches_a, unmatched_tracks_a, unmatched_detections = \
        linear_assignment.matching_cascade(
            gated_metric, self.metric.matching_threshold, self.max_age,
            self.tracks, detections, confirmed_tracks)

    # Associate remaining tracks together with unconfirmed tracks using IOU.
    # 将所有状态为未确定态的轨迹和刚刚没有匹配上的轨迹组合为iou_track_candidates，
    # 进行IoU的匹配
    iou_track_candidates = unconfirmed_tracks + [
        k for k in unmatched_tracks_a if
        self.tracks[k].time_since_update == 1] # 刚刚没有匹配上
    # 未匹配
    unmatched_tracks_a = [
        k for k in unmatched_tracks_a if
        self.tracks[k].time_since_update != 1] # 已经很久没有匹配上
    
    '''
    !!!!!!!!!!!
    IOU 匹配
    对级联匹配中还没有匹配成功的目标再进行IoU匹配
    !!!!!!!!!!!
    '''
    # 虽然和级联匹配中使用的都是min_cost_matching作为核心，
    # 这里使用的metric是iou cost和以上不同
    matches_b, unmatched_tracks_b, unmatched_detections = \
        linear_assignment.min_cost_matching(
            iou_matching.iou_cost, self.max_iou_distance, self.tracks,
            detections, iou_track_candidates, unmatched_detections)

    matches = matches_a + matches_b # 组合两部分match得到的结果
    
    unmatched_tracks = list(set(unmatched_tracks_a + unmatched_tracks_b))
    return matches, unmatched_tracks, unmatched_detections
```

对照如下图：

![32cc30b16a49b923515403c84ea5f0f0.png](https://runcoderhang.github.io/thumbnails/cb9bbd2a0b354644b2287caa28736ddb.png)

#### 级联匹配函数

级联匹配函数展开：

```python
def matching_cascade(
        distance_metric, max_distance, cascade_depth, tracks, detections,
        track_indices=None, detection_indices=None):
    # 级联匹配
    
    # 1. 分配track_indices和detection_indices
    if track_indices is None:
        track_indices = list(range(len(tracks)))
    if detection_indices is None:
        detection_indices = list(range(len(detections)))

    unmatched_detections = detection_indices
    matches = []
    # cascade depth = max age 默认为70
    for level in range(cascade_depth):
        if len(unmatched_detections) == 0:  # No detections left
            break

        track_indices_l = [
            k for k in track_indices
            if tracks[k].time_since_update == 1 + level
        ]
        if len(track_indices_l) == 0:  # Nothing to match at this level
            continue
        # 2. 级联匹配核心内容就是这个函数
        matches_l, _, unmatched_detections = \
            min_cost_matching(
                distance_metric, max_distance, tracks, detections,
                track_indices_l, unmatched_detections)
        matches += matches_l
    unmatched_tracks = list(set(track_indices) - set(k for k, _ in matches))
    return matches, unmatched_tracks, unmatched_detections
```

可以看到和伪代码是一致的，文章上半部分也有提到这部分代码。这部分代码中还有一个核心的函数 min_cost_matching ，这个函数可以接收不同的 distance_metric ，在级联匹配和IoU匹配中都有用到。


#### min_cost_matching 函数

```python
def min_cost_matching(
        distance_metric, max_distance, tracks, detections, track_indices=None,
        detection_indices=None):
    if track_indices is None:
        track_indices = np.arange(len(tracks))
    if detection_indices is None:
        detection_indices = np.arange(len(detections))

    if len(detection_indices) == 0 or len(track_indices) == 0:
        return [], track_indices, detection_indices  # Nothing to match.
    # -----------------------------------------
    # Gated_distance——>
    #       1. cosine distance
    #       2. 马氏距离
    # 得到代价矩阵
    # -----------------------------------------
    # iou_cost——>
    #       仅仅计算track和detection之间的iou距离
    # -----------------------------------------
    cost_matrix = distance_metric(
        tracks, detections, track_indices, detection_indices)
    # -----------------------------------------
    # gated_distance中设置距离中最高上限，
    # 这里最远距离实际是在deep sort类中的max_dist参数设置的
    # 默认max_dist=0.2， 距离越小越好
    # -----------------------------------------
    # iou_cost情况下，max_distance的设置对应tracker中的max_iou_distance,
    # 默认值为max_iou_distance=0.7
    # 注意结果是1-iou，所以越小越好
    # -----------------------------------------
    cost_matrix[cost_matrix > max_distance] = max_distance + 1e-5

    # 匈牙利算法或者KM算法
    row_indices, col_indices = linear_assignment(cost_matrix)

    matches, unmatched_tracks, unmatched_detections = [], [], []
    
    # 这几个for循环用于对匹配结果进行筛选，得到匹配和未匹配的结果
    for col, detection_idx in enumerate(detection_indices):
        if col not in col_indices:
            unmatched_detections.append(detection_idx)
    for row, track_idx in enumerate(track_indices):
        if row not in row_indices:
            unmatched_tracks.append(track_idx)
    for row, col in zip(row_indices, col_indices):
        track_idx = track_indices[row]
        detection_idx = detection_indices[col]
        if cost_matrix[row, col] > max_distance:
            unmatched_tracks.append(track_idx)
            unmatched_detections.append(detection_idx)
        else:
            matches.append((track_idx, detection_idx))
    # 得到匹配，未匹配轨迹，未匹配检测
    return matches, unmatched_tracks, unmatched_detections
```

注释中提到distance_metric是有两个的：

- 第一个是级联匹配中传入的distance_metric是gated_metric, 其内部核心是计算的表观特征的级联匹配。

```python
def gated_metric(tracks, dets, track_indices, detection_indices):
     # 功能： 用于计算track和detection之间的距离，代价函数
    #        需要使用在KM算法之前
    features = np.array([dets[i].feature for i in detection_indices])
    targets = np.array([tracks[i].track_id for i in track_indices])

    # 通过最近邻（余弦距离）计算出 cost_matrix （代价矩阵）
    cost_matrix = self.metric.distance(features, targets)

    # 计算指派后的成本矩阵（代价矩阵）
    cost_matrix = linear_assignment.gate_cost_matrix(
        self.kf, cost_matrix, tracks, dets, track_indices,
        detection_indices)

    return cost_matrix
```

- 第二个是IOU匹配中的iou_matching.iou_cost:

```python
# 虽然和级联匹配中使用的都是min_cost_matching作为核心，
# 这里使用的metric是iou cost和以上不同
matches_b, unmatched_tracks_b, unmatched_detections = \
    linear_assignment.min_cost_matching(
        iou_matching.iou_cost,
        self.max_iou_distance, 
        self.tracks,
        detections,
        iou_track_candidates,
        unmatched_detections)
```

iou_cost 代价很容易理解,用于计算 Track 和 Detection 之间的 IOU 距离矩阵。

```python
def iou_cost(tracks, detections, track_indices=None,
             detection_indices=None):
    # 计算track和detection之间的iou距离矩阵
    
    if track_indices is None:
        track_indices = np.arange(len(tracks))
    if detection_indices is None:
        detection_indices = np.arange(len(detections))

    cost_matrix = np.zeros((len(track_indices), len(detection_indices)))
    for row, track_idx in enumerate(track_indices):
        if tracks[track_idx].time_since_update > 1:
            cost_matrix[row, :] = linear_assignment.INFTY_COST
            continue

        bbox = tracks[track_idx].to_tlwh()
        candidates = np.asarray(
            [detections[i].tlwh for i in detection_indices])
        cost_matrix[row, :] = 1. - iou(bbox, candidates)
    return cost_matrix
```

END