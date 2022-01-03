---
title: 泳池项目实验记录（六）
tags: 
date: 2021-12-26 23:30:00
categories: 学习总结
---


本周主要工作是排查泳池项目代码出现bug的原因。首先，目前的bug是在对泳池人员进行多目标跟踪的过程中，前端有时会出现同一个目标有两个坐标点的现象。按照我们目前代码的逻辑，我们的算法只会对同一个目标进行匹配，然后输出的也是该目标的头或者身体的目标坐标点。

我一开始认为是匹配的算法放在跟踪结果后所产生的原因。因为我们目前的流程是：

$$
检测 \to 跟踪 \to 人头与人体匹配 \to 
\left\{\begin{matrix}
 未匹配
     \left\{\begin{matrix}
     人头坐标框
     \\ 人体坐标框

    \end{matrix}\right.
 \\
 \\ 已匹配 \to 人头坐标框
\end{matrix}\right.
$$

随后，我改变代码的逻辑，将匹配算法放在检测结果之后，然后就变成了：

$$
检测 \to 人头与人体匹配 \to 
\left\{\begin{matrix}
 未匹配
     \left\{\begin{matrix}
     人头坐标框
     \\ 人体坐标框

    \end{matrix}\right.
 \\
 \\ 已匹配 \to 人体坐标框
\end{matrix}\right.
\to 跟踪
$$

代码中需要注意的地方就是：跟踪算法所需要的输入结果是 tensor 格式数据，因此，我们需要对所到的的目标框进行格式转换，这样能够对齐目标跟踪的输入格式。

```python
def track_results(camera_ids, detection_results, names, tracker_list, img_list):
    all_outputs = []
    outputs = []
    for i in range(len(detection_results)):
        xywhs_list = []
        confs_list = []
        clss_list = []
        person_list = to_matching_v2(detection_results[i])
        for person in person_list:
            if person["type"] == "person":
                xywhs_list.append(person["xywh"][1].unsqueeze(0))
                confs_list.append(person["conf"][1].unsqueeze(0))
                clss_list.append(person["cls"][1].unsqueeze(0))
            else:
                xywhs_list.append(person["xywh"].unsqueeze(0))
                confs_list.append(person["conf"].unsqueeze(0))
                clss_list.append(person["cls"].unsqueeze(0))

        if len(xywhs_list) > 0:
            xywhs = torch.cat(xywhs_list, dim=0)
            confs = torch.cat(confs_list, dim=0)
            clss = torch.cat(clss_list, dim=0)
            match_detection_result = {"bbox": xywhs, "confs": confs, "clss": clss}

            outputs = tracker_list[i].track(match_detection_result, names, img_list[i])
        all_outputs.append(outputs)
    return all_outputs

def to_matching_v2(detection_result):
    head_list = []
    body_list = []
    head_indices = []
    body_indices = []
    person_list = []
    xywhs_list = detection_result["bbox"]
    confs_list = detection_result["confs"]
    clss_list = detection_result["clss"]
    if len(xywhs_list) > 0:
        id = 0
        match_idx = 0
        for (xywh, conf, cls) in zip(xywhs_list, confs_list, clss_list):
            bbox_x = int(xywh[0])
            bbox_y = int(xywh[1])
            bbox_w = int(xywh[2])
            bbox_h = int(xywh[3])
            c = int(cls)
            xmax = bbox_x + bbox_w // 2
            ymax = bbox_y + bbox_h // 2
            xmin = bbox_x - bbox_w // 2
            ymin = bbox_y - bbox_h // 2

            # 坐标按顺时针
            bbox = [[xmin, ymin],
                    [xmax, ymin],
                    [xmax, ymax],
                    [xmin, ymax]]

            # ---datasets: person  overwater underwater
            part_dict = {"index": match_idx, "bbox": bbox, "xywh": xywh, "conf": conf, "cls": cls}
            if c == 0:
                body_list.append(part_dict)
            elif c == 1 or c == 2:
                head_list.append(part_dict)
            id += 1
            match_idx += 1

        iou_matrix = np.zeros((len(head_list), len(body_list)))

        for i, head_points in enumerate(head_list):
            for j, body_points in enumerate(body_list):
                iou_matrix[i, j] = - intersection(head_points["bbox"], body_points["bbox"])
                # iou_matrix[i, j] = 1 - IOU(head_points, body_points)

        matched_row_indices, matched_col_indices = linear_sum_assignment(iou_matrix)
        iou_matrix = - iou_matrix

        idx = iou_matrix[matched_row_indices, matched_col_indices] > 0.95
        matched_row_indices = matched_row_indices[idx]
        matched_col_indices = matched_col_indices[idx]

        # 添加匹配上的 head 和 body
        for row_ind, col_ind in zip(matched_row_indices, matched_col_indices):
            head_indices.append(head_list[row_ind]["index"])
            body_indices.append(body_list[col_ind]["index"])
            head_list[row_ind]["bbox"] = to_center_with_corners(head_list[row_ind]["bbox"])
            body_list[col_ind]["bbox"] = to_center_with_corners(body_list[col_ind]["bbox"])
            person_list.append({"type": "person",
                                "bbox": [head_list[row_ind]["bbox"], body_list[col_ind]["bbox"]],
                                "xywh": [head_list[row_ind]["xywh"], body_list[col_ind]["xywh"]],
                                "conf": [head_list[row_ind]["conf"], body_list[col_ind]["conf"]],
                                "cls": [head_list[row_ind]["cls"], body_list[col_ind]["cls"]]})

        # 添加未匹配上的 head 或 body
        unmatch_idx = 0
        for (xywh, conf, cls) in zip(xywhs_list, confs_list, clss_list):
            if unmatch_idx not in head_indices and unmatch_idx not in body_indices:
                bbox_x = int(xywh[0])
                bbox_y = int(xywh[1])
                bbox_w = int(xywh[2])
                bbox_h = int(xywh[3])
                c = int(cls)
                bbox = [bbox_x, bbox_y, bbox_w, bbox_h]

                # ---datasets: person  overwater underwater
                if cls == 0:
                    person_list.append({"type": "body",
                                        "bbox": bbox,
                                        "xywh": xywh,
                                        "conf": conf,
                                        "cls": cls})
                else:
                    person_list.append({"type": "head",
                                        "bbox": bbox,
                                        "xywh": xywh,
                                        "conf": conf,
                                        "cls": cls})
            id += 1
            unmatch_idx += 1
    return person_list
```


