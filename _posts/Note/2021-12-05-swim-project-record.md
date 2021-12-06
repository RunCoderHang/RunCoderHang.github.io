---
title:  泳池项目实验记录（四）
tags: 
date: 2021-12-05 23:30:00
categories: 学习总结
---

本周主要完成了全泳池的跟踪以及溺水判断的功能。

全泳池溺水判断要比单个区域的溺水判断复杂一点，因为在全泳池溺水判断中我们加入了跟踪模块去对目标进行锁定。即加入跟踪模块后，我们就会基于每个目标一个 ID 值，然后根据这个 ID 进行溺水判断。

全泳池溺水判断中，我们需要记录大量的历史信息，并且这些历史信息都需要字典去保存。然后得到当前信息并与历史信息进行比对，最终我们就能够判断出该 ID 值的目标是否发生溺水。

代码如下：


```python
import time
class DrownPredict:
    def __init__(self, drown_duration=5, warning_duration=3, lose_duration=10):
        self.drown_duration = drown_duration
        self.warning_duration = warning_duration
        self.lose_duration = lose_duration

        # Plan B
        self.person_list = []
        self.person_dict = {}
        self.end = None

    # input: -1 under water, 0 no head, 1 on the water
    # return: -1 no drown, 1 warning, 2 drown
    def predict_v2(self, id, cls, present_ids):
        class_list = []
        record_ids = [person["id"] for person in self.person_list]  # 前面帧记录的所有目标id
        if id not in record_ids:
            class_list.append((cls, time.time()))
            lose_duration = 1
            self.person_dict = {"id": id, "class_list": class_list, "lose_duration": lose_duration}
            self.person_list.append(self.person_dict)
            return -1
        else:
            self.remove_lose_person(present_ids)
            return self.predict_state(id, cls)

    # 根据历史信息预测目标状态
    def predict_state(self, id, cls):
        for idx, person in enumerate(self.person_list):
            # 找到对应id的前几帧记录信息
            if person["id"] == id:
                person["class_list"].append((cls, time.time()))
                now = time.time()
                print(str(id) + ": start class list ", person["class_list"])
                person["class_list"] = [cls for cls in person["class_list"]
                                        if now - cls[1] < 15 + 1]
                print(str(id) + ": after class list ", person["class_list"])
                index = 0
                for cls in person["class_list"]:
                    if cls[0] == -1:
                        break
                    index += 1
                if index >= len(person["class_list"]) - 1:
                    print(str(id) + " no drown")
                    return -1
                temp_class_list = person["class_list"][index:-1]
                if temp_class_list[-1][0] > 0:
                    print(str(id) + " no drown")
                    return -1
                count = 0
                for cls in temp_class_list:
                    if cls[0] == 1:
                        count += 1
                if count < 3:
                    if temp_class_list[-1][1] - temp_class_list[0][1] > self.drown_duration:
                        print(str(id) + " drown")
                        return 2
                    if temp_class_list[-1][1] - temp_class_list[0][1] > self.warning_duration:
                        print(str(id) + " warning")
                        return 1
                print(str(id) + " no drown")
                return -1

    # 删除丢失的id的目标信息
    def remove_lose_person(self, present_ids):
        for idx, person in enumerate(self.person_list):
            # 删除丢失的id的目标信息
            if person["lose_duration"] > self.lose_duration:
                self.person_list.remove(person)

            # 记录丢失的id的持续时长
            if person["id"] not in present_ids:
                person["lose_duration"] += 1
            else:
                person["lose_duration"] = 1


```