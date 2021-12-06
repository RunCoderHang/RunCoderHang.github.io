---
title:  泳池项目实验记录（三）
tags: 
date: 2021-11-28 23:30:00
categories: 学习总结
---


本周完成了泳池内人头与人体的匹配算法，即：匈牙利匹配算法 + IOU

代码如下：

```python
def to_matching(outputs):
    head_list = []
    body_list = []
    head_indices = []
    body_indices = []
    person_list = []
    if len(outputs) > 0:
        for index, output in enumerate(outputs):
            id = int(output[4])
            cls = int(output[5])
            xmin, ymin, xmax, ymax = int(output[0]), int(output[1]), int(output[2]), int(output[3])
            # 坐标按顺时针
            bbox = [[xmin, ymin],
                    [xmax, ymin],
                    [xmax, ymax],
                    [xmin, ymax]]

            # ---datasets: head  body_overwater  body_underwater
            # if c == 0:
            #     head_list.append({"index": index, "id": id, "class": cls, "bbox": bbox})
            #     head_indices.append(index)
            # elif c == 1 or c == 2:
            #     body_list.append({"index": index, "id": id, "class": cls, "bbox": bbox})
            #     body_indices.append(index)

            # ---datasets: person  overwater underwater
            if cls == 0:
                body_list.append({"index": index, "id": id, "class": cls, "bbox": bbox})
            elif cls == 1 or cls == 2:
                head_list.append({"index": index, "id": id, "class": cls, "bbox": bbox})

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
            person_list.append({"type": "person", "id": body_list[col_ind]["id"], "class": head_list[row_ind]["class"],
                                "bbox": [head_list[row_ind]["bbox"], body_list[col_ind]["bbox"]]})

        # 添加未匹配上的 head 或 body
        for index, output in enumerate(outputs):
            if index not in head_indices and index not in body_indices:
                id = int(output[4])
                cls = int(output[5])
                xmin, ymin, xmax, ymax = int(output[0]), int(output[1]), int(output[2]), int(output[3])
                bbox = to_center_with_corner([xmin, ymin, xmax, ymax])

                # ---datasets: head  body_overwater  body_underwater
                # if cls == 0:
                #     person_list.append({"type": "head", "id": id, "bbox": bbox})
                # else:
                #     person_list.append({"type": "body", "id": id, "bbox": bbox})

                # ---datasets: person  overwater underwater
                if cls == 0:
                    person_list.append({"type": "body", "id": id, "class": cls, "bbox": bbox})
                else:
                    person_list.append({"type": "head", "id": id, "class": cls, "bbox": bbox})

    return person_list

```


