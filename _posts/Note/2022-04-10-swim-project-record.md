---
title: 泳池项目实验记录（十七）
tags: 
date: 2022-04-10 23:30:00
categories: 学习总结
---


根据上周的需求，新添加了对溺水人员所在的泳道进行判断。思路很简单：使用标注工具将每个泳道的所在区域进行划分，然后判断该溺水人员的坐标是否在这几个泳道的区域中。

一个坐标点是否在某个区域的判断就需要用到 `from shapely.geometry import Point, Polygon` 工具包。

首先得到每个相机下的每个泳道的区域，并存储至字典中。

```python
'''
input: json_path: 输入的 json_path 是包含单个相机下泳道的 mask
output: camera_lane_regions 单个相机下泳道的 Polygon 区域，如下所示：
 [
    {
        "lane_no": lane_1, 
        "lane_coords": mask_coords_1  # 泳道坐标所有点的原始数据
        "lane_region": mask_region_1  # 通过Polygon转换后的泳道区域
    },
    {
        "lane_no": lane_2, 
        "lane_coords": mask_coords_2
        "lane_region": mask_region_2
    },
    ...
]
'''
# 输入的 json_path 是包含单个相机下泳道的 mask
# 得到所有相机下泳道的 mask
def get_lane_regions_single_camera(json_path, camera_ip):
    mask_file = open(json_path, 'r')
    annotations = json.load(mask_file)
    camera_lane_regions = {}
    regions = annotations[camera_ip]["regions"]
    lane_regions = []
    for region in regions:
        lane_points_x = region["shape_attributes"]["all_points_x"]
        lane_points_y = region["shape_attributes"]["all_points_y"]
        lane_no = region["region_attributes"]["name"]
        mask_coords = []
        for x, y in zip(lane_points_x, lane_points_y):
            mask_point = (x, y)
            mask_coords.append(mask_point)
        mask_region = Polygon(mask_coords)
        lane_region = {"lane_no": lane_no, "lane_coords": mask_coords, "lane_region": mask_region}
        lane_regions.append(lane_region)
    return lane_regions
```

泳道区域判断算法：

输入：溺水人员的信息（type、id、类别、坐标框）
输出：溺水人员的信息（type、id、类别、坐标框、泳道编号）& 泳道编号

```python
'''
输入的 person 格式来自于 iou_match 的 track_to_matching
person (字典):
{
    "type": "person" / "head" / "body",
    "id": id,
    "class": cls,
    "bbox": [head_bbox, body_bbox] / head_bbox / body_bbox
}

lane_regions (列表): 其中一个相机的泳池泳道信息
[
    {
        "lane_no": lane, 
        "lane_coords": mask_coords  # 泳道坐标所有点的原始数据
        "lane_region": mask_region  # 通过Polygon转换后的泳道区域
    },
    ...
]
'''
def query_lane_region(warn_person, lane_regions):
    lane_no = 0  # 0 表示不是泳道编号，后面需要去除
    for lane in lane_regions:
        if warn_person["type"] == "person":
            warn_bbox = warn_person["bbox"][0]
        else:
            warn_bbox = warn_person["bbox"]
        point = Point(warn_bbox[0], warn_bbox[-1])
        if lane["lane_region"].contains(point):
            lane_no = int(lane["lane_no"].split("_")[1])
            warn_person["lane_no"] = lane_no
    return lane_no, warn_person
```


