---
title:  泳池项目实验记录（二）
tags: 
date: 2021-11-14 23:30:00
categories: 学习总结
---


这一周的工作是一直在配合泳池项目人员进行演示，并且调试溺水人员报警功能的程序。报警功能有三个重要的参数：drown_duration、warning_duration、count，分别代表的含义是：溺水报警持续时间、溺水预警持续时间、溺水报警判断的帧数。

溺水预测程序是使用一个队列记录目标的状态，而目标的状态无非是人头在水上和水下。我们使用这个队列去逐帧的记录该目标的人头是在水上还是在水下，随后我们可以根据人头在水下的时长去判断该目标是否溺水。当然，这个队列是不可能去存储所有帧的目标状态，我们会使用定时的时长去数据清洗，得到一段规定范围内的长度去记录该目标的状态。

下面代码就是我们数据清洗的功能：

```python
self.state_list.append((state, time.time()))
now = time.time()
self.state_list = [state for state in self.state_list if now - state[1] < self.drown_duration+1]
```

- drown_duration：虽说变量名为溺水报警时间，不过也担任着规定一定长度的队列功能。

在溺水预测程序中，我们还要去全面地考虑其他情况，例如目标沉入水中，相机检测不到头部，这样就会发生漏检。如果发生这种情况，我们就需要去判断该目标沉入水中的动作到底是溺水还是在潜泳。

因此，我们直接使用了 count 去计算该队列中到底有多少帧是该目标的头部是在水下的状态。即使发生了漏检、错检的情况，当我们依然可以通过计算一段时间内该目标头部在水下的时间长度，如果超过 warning_duration 则进行预警，如果超过 drown_duration 则进行报警。 

```python
if count < 4:
    if temp_state_list[-1][1]-temp_state_list[0][1] > self.drown_duration:
        print("drown")
        return 2
    if temp_state_list[-1][1]-temp_state_list[0][1] > self.warning_duration:
        print("warning")
        return 1
```


不过在本次测试中，我发现溺水人员和游泳人员本身在泳道上的移动距离是有所差别的。我在想，能否通过加上距离这部分作为一个权重去判断该人员是在游泳还是溺水。后面就需要去做大量的实验去调试好这部分功能。






