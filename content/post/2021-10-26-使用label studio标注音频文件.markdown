---
layout:     post
title:      使用label studio标注音频文件
subtitle:   label-studio标注音频文件
date:       2021-10-26
author:     Bingo
header-img: img/post-bg-label-studio.png
catalog: true
tags:
    - Machine Learning
    - Label Studio
    - ASR
---

> 最近有个小朋友接个任务，帮助把音频转写成文字，并进行标注。<br>
> 项目地址: [speech-labeled](https://github.com/yanhuibin315/speech-labeled)

# 前提
- 公共的可访问audio文件地址
- 腾讯云账号并开通音频转写权限
- 已经搭建好[Label Studio](https://labelstud.io/)

# 步骤
- 预转写: 最好有个基本模型能做到基本转写任务，此处选用腾讯云的语音转写模型
- 创建label task
- 人工校验


# 1. 预转写
经过多方对比，决定采用腾讯云的转写模型，使用腾讯云转写音频主要包括如下步骤：

## 1-1. 创建转写任务

```python
import json
from tencentcloud.common import credential
from tencentcloud.common.profile.client_profile import ClientProfile
from tencentcloud.common.profile.http_profile import HttpProfile
from tencentcloud.common.exception.tencent_cloud_sdk_exception import TencentCloudSDKException
from tencentcloud.asr.v20190614 import asr_client, models

def create_url_task(audio_url):
    SecretId = "******"
    SecretKey = "******"

    try:
        cred = credential.Credential(SecretId, SecretKey)
        httpProfile = HttpProfile()
        httpProfile.endpoint = "asr.tencentcloudapi.com"

        clientProfile = ClientProfile()
        clientProfile.httpProfile = httpProfile
        client = asr_client.AsrClient(cred, "", clientProfile)

        req = models.CreateRecTaskRequest()
        params = {
            "EngineModelType": "16k_zh",
            "ChannelNum": 1,
            "SpeakerDiarization": 1,  # 需要做说话人分割
            "SpeakerNumber": 2,
            "ResTextFormat": 0,
            "SourceType": 0,
            "Url": audio_url  # 音频地址
        }
        req.from_json_string(json.dumps(params))

        resp = client.CreateRecTask(req)
        return resp.to_json_string()

    except TencentCloudSDKException as err:
        print(err)
        return None
```

## 1-2. 获取转写任务结果
完成1后，会得到任务ID，再查询任务结果即可

```python
import json
from tencentcloud.common import credential
from tencentcloud.common.profile.client_profile import ClientProfile
from tencentcloud.common.profile.http_profile import HttpProfile
from tencentcloud.common.exception.tencent_cloud_sdk_exception import TencentCloudSDKException
from tencentcloud.asr.v20190614 import asr_client, models

def get_url_res(task_id):
    SecretId = "******"
    SecretKey = "******"
    try:
        cred = credential.Credential(SecretId, SecretKey)
        httpProfile = HttpProfile()
        httpProfile.endpoint = "asr.tencentcloudapi.com"

        clientProfile = ClientProfile()
        clientProfile.httpProfile = httpProfile
        client = asr_client.AsrClient(cred, "", clientProfile)

        req = models.DescribeTaskStatusRequest()
        params = {
            "TaskId": task_id
        }
        req.from_json_string(json.dumps(params))

        resp = client.DescribeTaskStatus(req)
        return resp.to_json_string()

    except TencentCloudSDKException as err:
        print(err)
        return None
```

结果大概长这样

```json
{
  "Data": {
    "TaskId": 12345678,
    "Status": 2,
    "StatusStr": "success",
    "Result": "[0:0.930,0:2.640,0]  哎，你好，哎。\n[0:2.640,0:4.650,1]  你好，我是自如中介。\n",
    "ErrorMsg": "",
    "ResultDetail": null
  },
  "RequestId": "qq-qq-qq-qq"
}

```

# 2. 创建标注项目
## 2-1. Project Name
- Project Name: wav-to-text
- Description: blablabla
## 2-2. Data Import
这里有两种上传方式：
1. add url: 粘贴公开的音频地址
2. Upload files: 把本地的音频文件上传
## 2-3. Labeling Setup
因为我们需要用到说话人标记，此处选择：

Audio/Speech Processing -> Automatic Speech Recognition using Segments

![](https://raw.githubusercontent.com/yanhuibin315/yanhuibin315.github.io/master/img/label-studio-1.png)
## 2-4. 确定标签
在标签中加上“Speaker-0”, "Speaker-1"

# 3. 创建标注注解
- 解析tencent转写结果
- 向label-studio发送更新annotation请求

```python
"""
更新label studio的标注结果
"""
import requests
import json
import uuid

def profile_format(input):
    if int(input) == 0:
        return "Speaker-0"
    else:
        return "Speaker-1"

def time_format(input):
    """
    解析时间标签
    :param input: 0:0.930
    :return: 0.930
    """
    if len(input.split(':')) != 2:
        print("Time Invalid!")
        return ""
    min, sec = input.split(':')[0], input.split(':')[1]
    res = int(min) * 60 + float(sec)
    return str(res)

def analystic(res_file):
    """
    解析转写结果
    :param res_file:
    :return:
    """
    results = []
    original_length = 0
    with open(res_file, "r") as f:
        response = json.loads(f.read())
    Result = response["Data"]["Result"].split("\n")
    for res in Result:
        if len(res.split('  ')) != 2: continue
        uid = uuid.uuid4()
        time, text = res.split('  ')
        time = time.replace('[', '').replace(']', '')
        start, end, profile = time.split(',')
        start = time_format(start)
        end = time_format(end)
        profile = profile_format(profile)
        original_length = end
        results.extend([{
            "original_length": original_length,
            "value": {
                "start": float(start),
                "end": float(end),
                "labels": [
                    profile
                ]
            },
            "id": f"wavesufer_{uid}",
            "from_name": "labels",
            "to_name": "audio",
            "type": "labels"
        },{
            "original_length": original_length,
            "value": {
                "start": float(start),
                "end": float(end),
                "text": [
                    text
                ]
            },
            "id": f"wavesufer_{uid}",
            "from_name": "transcription",
            "to_name": "audio",
            "type": "textarea"
        }])

    for res in results:
        res["original_length"] = original_length
    return results




url = "http://localhost:8080/api/tasks/3/annotations/"

headers = {
  'Authorization': 'Token ********',
  'Content-Type': 'application/json',
}

res_file = "response.json"
data = dict()
data["result"] = analystic(res_file)
data["was_cancelled"] = True
data["ground_truth"] = True
data["lead_time"] = 30  # 耗时
data["task"] = 3  # 任务ID


payload = json.dumps(data)
response = requests.request("PATCH", url, headers=headers, data=payload)

print(response.text)

```

# 4. 人工审核
然后再在UI界面上审核标注即可
![](https://raw.githubusercontent.com/yanhuibin315/yanhuibin315.github.io/master/img/label-studio-2.png)