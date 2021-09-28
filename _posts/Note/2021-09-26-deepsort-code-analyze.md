---
title:  DeepSORT之代码解析
tags: 
- DeepSORT
date: 2021-09-26 23:30:00
categories: 学习总结
---

在《DEEP LEARNING IN VIDEO MULTI-OBJECT TRACKING: A SURVEY》这篇基于深度学习的多目标跟踪的综述中，描述了MOT问题中四个主要步骤：

- 给定视频原始帧。
- 运行目标检测器如 Faster R-CNN、YOLOv3、SSD 等进行检测，获取目标检测框。
- 将所有目标框中对应的目标抠出来，进行特征提取（包括表观特征或者运动特征）。
- 进行相似度计算，计算前后两帧目标之间的匹配程度（前后属于同一个目标的之间的距离比较小，不同目标的距离比较大）
- 数据关联，为每个对象分配目标的ID。

以上就是四个核心步骤，其中核心是检测，SORT论文的摘要中提到，仅仅换一个更好的检测器，就可以将目标跟踪表现提升18.9%。


# DeepSORT

DeepSORT 中最大的特点是加入外观信息，借用了 ReID 领域模型来提取特征，减少了ID switch的次数。整体流程图如下：

![32cc30b16a49b923515403c84ea5f0f0.png](https://runcoderhang.github.io/thumbnails/cb9bbd2a0b354644b2287caa28736ddb.png)

可以看出，Deep SORT算法在SORT算法的基础上增加了级联匹配(Matching Cascade)+新轨迹的确认(confirmed)。总体流程就是：

- 卡尔曼滤波器预测轨迹Tracks
- 使用匈牙利算法将预测得到的轨迹Tracks和当前帧中的detections进行匹配(级联匹配和IOU匹配)
- 卡尔曼滤波更新。

其中上图中的级联匹配展开如下：

![170d0c956eb37063f6bfef7c1703fcf2.png](https://runcoderhang.github.io/thumbnails/bc08bf888fb94d1db372f628dcb6756a.png)

上图非常清晰地解释了如何进行级联匹配，上图由虚线划分为两部分：

上半部分中计算相似度矩阵的方法使用到了 **外观模型**(ReID) 和 **运动模型**(马氏距离) 来计算相似度，得到代价矩阵，另外一个则是门控矩阵，用于限制代价矩阵中过大的值。


下半部分中是是 **级联匹配的数据关联** 步骤，匹配过程是一个循环(max age个迭代，默认为70)，也就是从missing age=0到missing age=70的轨迹和Detections进行匹配，没有丢失过的轨迹优先匹配，丢失较为久远的就靠后匹配。通过这部分处理，可以重新将被遮挡目标找回，降低被遮挡然后再出现的目标发生的ID Switch次数。

将 Detection 和 Track 进行匹配，所以出现几种情况

1. Detection 和 Track 匹配，也就是 Matched Tracks 。普通连续跟踪的目标都属于这种情况，前后两帧都有目标，能够匹配上。
2. Detection 没有找到匹配的 Track ，也就是 Unmatched Detections 。图像中突然出现新的目标的时候， Detection 无法在之前的 Track 找到匹配的目标。
3. Track 没有找到匹配的 Detection ，也就是 Unmatched Tracks 。连续追踪的目标超出图像区域， Track 无法与当前任意一个 Detection 匹配。
4. 以上没有涉及一种特殊的情况，就是两个目标遮挡的情况。刚刚被遮挡的目标的 Track 也无法匹配 Detection ，目标暂时从图像中消失。之后被遮挡目标再次出现的时候，应该尽量让被遮挡目标分配的ID不发生变动，减少 ID Switch 出现的次数，这就需要用到级联匹配了。

# DeepSORT代码解析

![023e88e771efb45eaa55d8d757a5d341.png](https://runcoderhang.github.io/thumbnails/d120fb3005e344c4b726211a1b38b6cf.png)

上图是 Github 库中有关 Deep SORT 的核心代码，不包括 Faster R-CNN 检测部分，所以主要将讲解这部分的几个文件

- DeepSORT 类
    - Deep ReID
        
        - Extractor 类
    - 跟踪器
        - Tracker 类
        - Detection 类
        - NearestNerighborDistanceMetric 类
        - gated_metric 类
        - KalmanFilter 类
    - 轨迹
        - Track 类
        - TrackState 类
        
        

DeepSort是核心类，调用其他模块，大体上可以分为三个模块：

- ReID模块，用于提取表观特征，原论文中是生成了128维的embedding。
- Track模块，轨迹类，用于保存一个Track的状态信息，是一个基本单位。
- Tracker模块，Tracker模块掌握最核心的算法，卡尔曼滤波和匈牙利算法都是通过调用这个模块来完成的。

DeepSort类对外接口非常简单：

```python
self.deepsort = DeepSort(args.deepsort_checkpoint)#实例化
outputs = self.deepsort.update(bbox_xcycwh, cls_conf, im)#通过接收目标检测结果进行更新
```

在外部调用的时候只需要以上两步即可，非常简单。

通过类图，对整体模块有了框架上理解，下面深入理解一下这些模块。

## 核心模块

### Detection 类

```python
class Detection(object):
    """
    This class represents a bounding box detection in a single image.
 """
    def __init__(self, tlwh, confidence, feature):
        self.tlwh = np.asarray(tlwh, dtype=np.float)
        self.confidence = float(confidence)
        self.feature = np.asarray(feature, dtype=np.float32)
    def to_tlbr(self):
        """Convert bounding box to format `(min x, min y, max x, max y)`, i.e.,
        `(top left, bottom right)`.
        """
        ret = self.tlwh.copy()
        ret[2:] += ret[:2]
        return ret
    def to_xyah(self):
        """Convert bounding box to format `(center x, center y, aspect ratio,
        height)`, where the aspect ratio is `width / height`.
        """
        ret = self.tlwh.copy()
        ret[:2] += ret[2:] / 2
        ret[2] /= ret[3]
        return ret
```

Detection 类用于保存通过目标检测器得到的一个检测框，包含 top left 坐标+框的宽和高，以及该 bbox 的置信度还有通过 reid 获取得到的对应的 embedding 。除此以外提供了不同 bbox 位置格式的转换方法：

- tlwh: 代表左上角坐标+宽高
- tlbr: 代表左上角坐标+右下角坐标
- xyah: 代表中心坐标+宽高比+高


### Track 类


```python
class Track:
    # 一个轨迹的信息，包含(x,y,a,h) & v
    """
    A single target track with state space `(x, y, a, h)` and associated
    velocities, where `(x, y)` is the center of the bounding box, `a` is the
    aspect ratio and `h` is the height.
    """

    def __init__(self, mean, covariance, track_id, n_init, max_age,
                 feature=None):
        # max age是一个存活期限，默认为70帧,在
        self.mean = mean
        self.covariance = covariance
        self.track_id = track_id
        self.hits = 1 
        # hits和n_init进行比较
        # hits每次update的时候进行一次更新（只有match的时候才进行update）
        # hits代表匹配上了多少次，匹配次数超过n_init就会设置为confirmed状态
        self.age = 1 # 没有用到，和time_since_update功能重复
        self.time_since_update = 0
        # 每次调用predict函数的时候就会+1
        # 每次调用update函数的时候就会设置为0

        self.state = TrackState.Tentative
        self.features = []
        # 每个track对应多个features, 每次更新都将最新的feature添加到列表中
        if feature is not None:
            self.features.append(feature)

        self._n_init = n_init  # 如果连续n_init帧都没有出现失配，设置为deleted状态
        self._max_age = max_age  # 上限
```

Track类主要存储的是轨迹信息，mean和covariance是保存的框的位置和速度信息，track_id代表分配给这个轨迹的ID。state代表框的状态，有三种：

- Tentative: 不确定态，这种状态会在初始化一个Track的时候分配，并且只有在连续匹配上n_init帧才会转变为确定态。如果在处于不确定态的情况下没有匹配上任何detection，那将转变为删除态。
- Confirmed: 确定态，代表该Track确实处于匹配状态。如果当前Track属于确定态，但是失配连续达到max age次数的时候，就会被转变为删除态。
- Deleted: 删除态，说明该Track已经失效。

![d89e92ffacd291f90e3a55046d12781c.png](https://runcoderhang.github.io/thumbnails/c0c2fad8e2a54552a5c6c0f16486797a.png)

- max_age 代表一个 Track 存活期限，他需要和 time_since_update 变量进行比对。
- time_since_update 是每次轨迹调用 predict 函数的时候就会 +1，每次调用 predict 的时候就会重置为0，也就是说如果一个轨迹长时间没有 update (没有匹配上)的时候，就会不断增加，直到 time_since_update 超过 max_age (默认70)，将这个 Track 从 Tracker 中的列表删除。
- hits 代表连续确认多少次，用在从不确定态转为确定态的时候。每次 Track 进行 update 的时候， hits 就会 +1, 如果 hits>n_init (默认为3)，也就是连续三帧的该轨迹都得到了匹配，这时候才将不确定态转为确定态。


### ReID特征提取部分

ReID 网络是独立于目标检测和跟踪器的模块，功能是提取对应 bounding box 中的 feature ,得到一个固定维度的 embedding 作为该 bbox 的代表，供计算相似度时使用。

```python
class Extractor(object):
    def __init__(self, model_name, model_path, use_cuda=True):
        self.net = build_model(name=model_name,
                               num_classes=96)
        self.device = "cuda" if torch.cuda.is_available(
        ) and use_cuda else "cpu"
        state_dict = torch.load(model_path)['net_dict']
        self.net.load_state_dict(state_dict)
        print("Loading weights from {}... Done!".format(model_path))
        self.net.to(self.device)
        self.size = (128,128)
        self.norm = transforms.Compose([
            transforms.ToTensor(),
            transforms.Normalize([0.3568, 0.3141, 0.2781],
                                 [0.1752, 0.1857, 0.1879])
        ])

    def _preprocess(self, im_crops):
        """
        TODO:
            1. to float with scale from 0 to 1
            2. resize to (64, 128) as Market1501 dataset did
            3. concatenate to a numpy array
            3. to torch Tensor
            4. normalize
        """
        def _resize(im, size):
            return cv2.resize(im.astype(np.float32) / 255., size)

        im_batch = torch.cat([
            self.norm(_resize(im, self.size)).unsqueeze(0) for im in im_crops
        ],dim=0).float()
        return im_batch

    def __call__(self, im_crops):
        im_batch = self._preprocess(im_crops)
        with torch.no_grad():
            im_batch = im_batch.to(self.device)
            features = self.net(im_batch)
        return features.cpu().numpy()
```

模型训练是按照传统 ReID 的方法进行，使用 Extractor 类的时候输入为一个 list 的图片，得到图片对应的特征。

### NearestNeighborDistanceMetric类

这个类中用到了两个计算距离的函数：

1. 计算欧氏距离

```python
def _pdist(a, b):
    # 用于计算成对的平方距离
    # a NxM 代表N个对象，每个对象有M个数值作为embedding进行比较
    # b LxM 代表L个对象，每个对象有M个数值作为embedding进行比较
    # 返回的是NxL的矩阵，比如dist[i][j]代表a[i]和b[j]之间的平方和距离
    # 实现见：https://blog.csdn.net/frankzd/article/details/80251042
    a, b = np.asarray(a), np.asarray(b)  # 拷贝一份数据
    if len(a) == 0 or len(b) == 0:
        return np.zeros((len(a), len(b)))
    a2, b2 = np.square(a).sum(axis=1), np.square(
        b).sum(axis=1)  # 求每个embedding的平方和
    # sum(N) + sum(L) -2 x [NxM]x[MxL] = [NxL]
    r2 = -2. * np.dot(a, b.T) + a2[:, None] + b2[None, :]
    r2 = np.clip(r2, 0., float(np.inf))
    return r2
```

2. 计算余弦距离


$$
\cos{(\theta)} = \frac{\sum_{i-1}^n{x_{1i} \ x_{2i}}}{\sqrt{\sum_{k=1}^n{x_{1i}}^2} \ \sqrt{\sum_{k=1}^n{x_{2i}}^2}}
$$

```python
def _cosine_distance(a, b, data_is_normalized=False):
    # a和b之间的余弦距离
    # a : [NxM] b : [LxM]
    # 余弦距离 = 1 - 余弦相似度
    # https://blog.csdn.net/u013749540/article/details/51813922
    if not data_is_normalized:
        # 需要将余弦相似度转化成类似欧氏距离的余弦距离。
        a = np.asarray(a) / np.linalg.norm(a, axis=1, keepdims=True)
        #  np.linalg.norm 操作是求向量的范式，默认是L2范式，等同于求向量的欧式距离。
        b = np.asarray(b) / np.linalg.norm(b, axis=1, keepdims=True)
    return 1. - np.dot(a, b.T)

```

注意：余弦距离 = 1 - 余弦相似度。


### Tracker 类

Tracker类是最核心的类，Tracker中保存了所有的轨迹信息，负责初始化第一帧的轨迹、卡尔曼滤波的预测和更新、负责级联匹配、IOU匹配等等核心工作。

```python
class Tracker:
    # 是一个多目标tracker，保存了很多个track轨迹
    # 负责调用卡尔曼滤波来预测track的新状态+进行匹配工作+初始化第一帧
    # Tracker调用update或predict的时候，其中的每个track也会各自调用自己的update或predict
    """
    This is the multi-target tracker.
    """

    def __init__(self, metric, max_iou_distance=0.7, max_age=70, n_init=3):
        # 调用的时候，后边的参数全部是默认的
        self.metric = metric 
        # metric是一个类，用于计算距离(余弦距离或马氏距离)
        self.max_iou_distance = max_iou_distance
        # 最大iou，iou匹配的时候使用
        self.max_age = max_age
        # 直接指定级联匹配的cascade_depth参数
        self.n_init = n_init
        # n_init代表需要n_init次数的update才会将track状态设置为confirmed

        self.kf = kalman_filter.KalmanFilter()# 卡尔曼滤波器
        self.tracks = [] # 保存一系列轨迹
        self._next_id = 1 # 下一个分配的轨迹id
 def predict(self):
        # 遍历每个track都进行一次预测
        """Propagate track state distributions one time step forward.

        This function should be called once every time step, before `update`.
        """
        for track in self.tracks:
            track.predict(self.kf)
```

然后来看最核心的 update 函数和 match 函数，可以对照下面的流程图一起看：

**update函数**

```python
def update(self, detections):
    # 进行测量的更新和轨迹管理
    """Perform measurement update and track management.

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
        # track更新对应的detection
        self.tracks[track_idx].update(self.kf, detections[detection_idx])

    # 2. 针对未匹配的tracker,调用mark_missed标记
    # track失配，若待定则删除，若update时间很久也删除
    # max age是一个存活期限，默认为70帧
    for track_idx in unmatched_tracks:
        self.tracks[track_idx].mark_missed()

    # 3. 针对未匹配的detection， detection失配，进行初始化
    for detection_idx in unmatched_detections:
        self._initiate_track(detections[detection_idx])

    # 得到最新的tracks列表，保存的是标记为confirmed和Tentative的track
    self.tracks = [t for t in self.tracks if not t.is_deleted()]

    # Update distance metric.
    active_targets = [t.track_id for t in self.tracks if t.is_confirmed()]
    # 获取所有confirmed状态的track id
    features, targets = [], []
    for track in self.tracks:
        if not track.is_confirmed():
            continue
        features += track.features  # 将tracks列表拼接到features列表
        # 获取每个feature对应的track id
        targets += [track.track_id for _ in track.features]
        track.features = []

    # 距离度量中的 特征集更新
    self.metric.partial_fit(np.asarray(features), np.asarray(targets),
                            active_targets)
```

**match函数**

```python
def _match(self, detections):
    # 主要功能是进行匹配，找到匹配的，未匹配的部分
    def gated_metric(tracks, dets, track_indices, detection_indices):
        # 功能： 用于计算track和detection之间的距离，代价函数
        #        需要使用在KM算法之前
        # 调用：
        # cost_matrix = distance_metric(tracks, detections,
        #                  track_indices, detection_indices)
        features = np.array([dets[i].feature for i in detection_indices])
        targets = np.array([tracks[i].track_id for i in track_indices])

        # 1. 通过最近邻计算出代价矩阵 cosine distance
        cost_matrix = self.metric.distance(features, targets)
        # 2. 计算马氏距离,得到新的状态矩阵
        cost_matrix = linear_assignment.gate_cost_matrix(
            self.kf, cost_matrix, tracks, dets, track_indices,
            detection_indices)
        return cost_matrix

    # Split track set into confirmed and unconfirmed tracks.
    # 划分不同轨迹的状态
    confirmed_tracks = [
        i for i, t in enumerate(self.tracks) if t.is_confirmed()
    ]
    unconfirmed_tracks = [
        i for i, t in enumerate(self.tracks) if not t.is_confirmed()
    ]

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
            gated_metric,
            self.metric.matching_threshold,
            self.max_age,
            self.tracks,
            detections,
            confirmed_tracks)

    # 将所有状态为未确定态的轨迹和刚刚没有匹配上的轨迹组合为iou_track_candidates，
    # 进行IoU的匹配
    iou_track_candidates = unconfirmed_tracks + [
        k for k in unmatched_tracks_a
        if self.tracks[k].time_since_update == 1  # 刚刚没有匹配上
    ]
    # 未匹配
    unmatched_tracks_a = [
        k for k in unmatched_tracks_a
        if self.tracks[k].time_since_update != 1  # 已经很久没有匹配上
    ]

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
            iou_matching.iou_cost,
            self.max_iou_distance,
            self.tracks,
            detections,
            iou_track_candidates,
            unmatched_detections)

    matches = matches_a + matches_b  # 组合两部分match得到的结果

    unmatched_tracks = list(set(unmatched_tracks_a + unmatched_tracks_b))
    return matches, unmatched_tracks, unmatched_detections
```

### 级联匹配

下边是论文中给出的级联匹配的伪代码：

![a46ab828f7d7b4564a4709147579efd2.png](https://runcoderhang.github.io/thumbnails/58959fc2ef98449fa161e90992f05e5c.png)

以下代码是伪代码对应的实现

```python
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
        min_cost_matching(  # max_distance=0.2
            distance_metric, max_distance, tracks, detections,
            track_indices_l, unmatched_detections)
    matches += matches_l
unmatched_tracks = list(set(track_indices) - set(k for k, _ in matches))
```

### 门控矩阵

门控矩阵的作用就是通过计算卡尔曼滤波的状态分布和测量值之间的距离对代价矩阵进行限制。

代价矩阵中的距离是 Track 和 Detection 之间的表观相似度，假如一个轨迹要去匹配两个表观特征非常相似的 Detection ，这样就很容易出错，但是这个时候分别让两个 Detection 计算与这个轨迹的马氏距离，并使用一个阈值 gating_threshold 进行限制，所以就可以将马氏距离较远的那个 Detection 区分开，可以降低错误的匹配。

```python
def gate_cost_matrix(
        kf, cost_matrix, tracks, detections, track_indices, detection_indices,
        gated_cost=INFTY_COST, only_position=False):
    # 根据通过卡尔曼滤波获得的状态分布，使成本矩阵中的不可行条目无效。
    gating_dim = 2 if only_position else 4
    gating_threshold = kalman_filter.chi2inv95[gating_dim]  # 9.4877

    measurements = np.asarray([detections[i].to_xyah()
                               for i in detection_indices])
    for row, track_idx in enumerate(track_indices):
        track = tracks[track_idx]
        gating_distance = kf.gating_distance(
            track.mean, track.covariance, measurements, only_position)
        cost_matrix[row, gating_distance >
                    gating_threshold] = gated_cost  # 设置为inf
    return cost_matrix
```

### 卡尔曼滤波器

![704d58371912ef751e649d3cef5f38a9.png](https://runcoderhang.github.io/thumbnails/d59dbeb9bbe745888e9193ac2bf3a334.png)

- 卡尔曼滤波首先根据当前帧(time=t)的状态进行 **预测** ，得到预测下一帧的状态(time=t+1)
- 得到测量结果，在 Deep SORT 中对应的测量就是 Detection ，即目标检测器提供的检测框。
- 将预测结果和测量结果进行 **更新** 。

**预测** 分两个公式：

第一个公式：

$$
{x}' = Fx
$$

其中 $F$ 为状态转移矩阵

第二个公式：

$$
{P}' = FPF^T + Q
$$

$P$ 是当前帧(time=t)的协方差， $Q$ 是卡尔曼滤波器的运动估计误差，代表不确定程度。

```python
def predict(self, mean, covariance):
    # 相当于得到t时刻估计值
    # Q 预测过程中噪声协方差
    std_pos = [
        self._std_weight_position * mean[3],
        self._std_weight_position * mean[3],
        1e-2,
        self._std_weight_position * mean[3]]

    std_vel = [
        self._std_weight_velocity * mean[3],
        self._std_weight_velocity * mean[3],
        1e-5,
        self._std_weight_velocity * mean[3]]

    # np.r_ 按列连接两个矩阵
    # 初始化噪声矩阵Q
    motion_cov = np.diag(np.square(np.r_[std_pos, std_vel]))

    # x' = Fx
    mean = np.dot(self._motion_mat, mean)

    # P' = FPF^T+Q
    covariance = np.linalg.multi_dot((
        self._motion_mat, covariance, self._motion_mat.T)) + motion_cov

    return mean, covariance
```

**更新**

$$
y = z - H{x}'
$$
$$
S = H{P}'H^T + R
$$
$$
K = {P}'H^TS^{-1}
$$
$$
x = {x}' + Ky
$$
$$
P = (I - KH){P}'
$$

```python
def project(self, mean, covariance):
    # R 测量过程中噪声的协方差
    std = [
        self._std_weight_position * mean[3],
        self._std_weight_position * mean[3],
        1e-1,
        self._std_weight_position * mean[3]]

    # 初始化噪声矩阵R
    innovation_cov = np.diag(np.square(std))

    # 将均值向量映射到检测空间，即Hx'
    mean = np.dot(self._update_mat, mean)

    # 将协方差矩阵映射到检测空间，即HP'H^T
    covariance = np.linalg.multi_dot((
        self._update_mat, covariance, self._update_mat.T))

    return mean, covariance + innovation_cov

def update(self, mean, covariance, measurement):
    # 通过估计值和观测值估计最新结果

    # 将均值和协方差映射到检测空间，得到 Hx' 和 S
    projected_mean, projected_cov = self.project(mean, covariance)

    # 矩阵分解
    chol_factor, lower = scipy.linalg.cho_factor(
        projected_cov, lower=True, check_finite=False)

    # 计算卡尔曼增益K
    kalman_gain = scipy.linalg.cho_solve(
        (chol_factor, lower), np.dot(covariance, self._update_mat.T).T,
        check_finite=False).T

    # z - Hx'
    innovation = measurement - projected_mean

    # x = x' + Ky
    new_mean = mean + np.dot(innovation, kalman_gain.T)

    # P = (I - KH)P'
    new_covariance = covariance - np.linalg.multi_dot((
        kalman_gain, projected_cov, kalman_gain.T))
    return new_mean, new_covariance
```

$$
y = z - H{x}'
$$

这个公式中，$z$ 是 Detection 的 mean ，不包含变化值，状态为 $[c_x,c_y,a,h]$ 。 $H$ 是测量矩阵，将 Track 的均值向量 ${x}'$ 映射到检测空间。计算的 $y$ 是 Detection 和 Track 的均值误差。

$$
S = H{P}'H^T + R
$$

$R$ 是目标检测器的噪声矩阵，是一个 4x4 的对角矩阵。 对角线上的值分别为中心点两个坐标以及宽高的噪声。

$$
K = {P}'H^TS^{-1}
$$

计算的是卡尔曼增益，是作用于衡量估计误差的权重。

$$
x = {x}' + Ky
$$

更新后的均值向量 $x$ 。

$$
P = (I - KH){P}'
$$

更新后的协方差矩阵。