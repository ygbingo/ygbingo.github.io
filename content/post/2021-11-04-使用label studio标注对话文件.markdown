---
layout:     post
title:      使用label studio标注对话文件
subtitle:   label-studio标注对话文件
date:       2021-11-04
author:     Bingo
header-img: img/post-bg-1104.png
catalog: true
tags:
    - Machine Learning
    - Label Studio
    - Dialogue
---

> 最近有个小朋友接个任务，已经收集了一些对话，需要对里面的对话内容打标，提取出质量较好的对话。<br>

# 前提
- 已有的对话样本
- 已经搭建好[Label Studio](https://labelstud.io/)

# 步骤
- 创建项目并指定label template
- 创建label task
- 标注
- 导出标注结果


# 1. 创建项目并指定label template

### 1-1. 通过可视化界面创建

- Create Project
- Input Project Name
- Labeling Setup: Conversational AI -> Intent Classification and Slot Filling
- Set Choice: Good/Bad/Neutral

![](https://raw.githubusercontent.com/yanhuibin315/yanhuibin315.github.io/master/img/post-1104-create-template.png)

### 1-2. 也可以通过code创建

```xml

<View>
  <HyperText name="dialog" value="$dialogs"/>
  <Header value="Rate last answer"/>
  <Choices name="rating" choice="single-radio" toName="dialog" showInline="true">
    <Choice value="Bad answer"/>
    <Choice value="Neutral answer"/>
    <Choice value="Good answer"/>
  </Choices>
  <Header value="Write your answer and press Enter"/>
  <TextArea toName="dialog" name="answer"/>
</View>

```

# 2. 创建label task

**假设已经有了几个待标注的对话样本：**

```jsonl
["还没睡咩", "没有你呢", "我也是", "怎么办阿", "有点困了", "睡不着阿"]
["大点儿声", "听不见", "大点声", "要多大声啊", "明天", "后天还有后天"]
["祝融号火星车此时此刻在火星上做什么呢", "这句话我没法接…"]
["大一点声", "大声叫", "小声点", "说话大声点", "小声点", "太小声！"]
```

**通过api的形式创建label task**

```python
import jsonlines
import requests
import json

PROJECT_ID = 8  # 此处使用自己的项目id

url = f"http://localhost:8080/api/projects/{PROJECT_ID}/import"

headers = {
  'Authorization': 'Token ******',
  'Content-Type': 'application/json',
}

reader = jsonlines.open("dialogue.jsonl", "r").iter()

idx = 0
MAX_TASK_NUM = 10
datas = []

for i in range(0, 1):
    while reader:
        data = dict()
        data["dialogs"] = json.dumps(next(reader), ensure_ascii=False)
        datas.append(data)

        idx += 1
        if idx >= MAX_TASK_NUM: break
    payload = json.dumps(datas, ensure_ascii=False).encode("utf-8")
    response = requests.request("POST", url, headers=headers, data=payload)
    print(response.text)
print("ALL task created!")

```



# 3. 标注

![](https://raw.githubusercontent.com/yanhuibin315/yanhuibin315.github.io/master/img/post-1104-label-task.png)


# 4. 导出标注结果

直接点击项目右上角：Export -> Json 即可！

