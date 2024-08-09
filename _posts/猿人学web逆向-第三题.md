---
title: 猿人学web逆向-第三题
tags: 
- 逆向
categories:
- 爬虫
---

目标链接：https://match.yuanrenxue.cn/match/3

### 0x01 初步分析

 找到目标接口，发现每次请求都会先请求一次jssm这个接口，

 另外分析接口，没有什么特殊的参数，只有一个session_id比较可疑，

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/undefined20240809152758.png)

先模拟直接请求，返回一段js代码，放到控制台看看，没什么用应该是干扰项，

然后再加上jssm请求，依旧返回的是js代码，

将这两次请求放到compare里面对比一下，cookie一致，headers也基本一致。。。

同样的请求，为什么放在浏览器能请求成功，改成python请求就不行了呢？是不是抓包有问题？

用fiddler抓包，

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/undefined20240809153744.png)

看起来没什么不同的地方，在fiddler中Composer模拟请求一下，

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/undefined20240809154134.png)

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/undefined20240809154208.png)

在fiddler模拟请求也是正常的，用pyhton模拟fiddler中的参数，

发现fiddler中正常请求的参数，和从浏览器中copy as curl的header顺序是不一样的，

难道是headers参数的问题，

将fiddler的参数全部copy下来，使用session请求，果不其然，拿到了正确的结果。



### 0x02 最终代码

// yrx3.py

```python
import requests
session = requests.session()


session.headers = {
    "Host": "match.yuanrenxue.cn",
    "Connection": "keep-alive",
    "Content-Length": "0",
    "Pragma": "no-cache",
    "Cache-Control": "no-cache",
    "sec-ch-ua": '"Not/A)Brand";v="8", "Chromium";v="126", "Google Chrome";v="126"',
    "sec-ch-ua-mobile": "?0",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36",
    "sec-ch-ua-platform": '"Windows"',
    "Accept": "*/*",
    "Origin": "https://match.yuanrenxue.cn",
    "Sec-Fetch-Site": "same-origin",
    "Sec-Fetch-Mode": "cors",
    "Sec-Fetch-Dest": "empty",
    "Referer": "https://match.yuanrenxue.cn/match/3",
    "Accept-Encoding": "gzip, deflate, br, zstd",
    "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
    "Cookie": "HMACCOUNT=5A4E07F0350148A2; Hm_lvt_9bcbda9cbf86757998a2339a0437208e=1721271896,1721306800; no-alert3=true; Hm_lvt_c99546cf032aaa5a679230de9a95c7db=1721271896,1722843351; m=465847b80dfa164eabe2eb66fb1ba7a7|1723100805000; tk=-3600877825130472532; sessionid=kaugd3f2royqiy7s04jhq3ujjy9l75a7; Hm_lpvt_9bcbda9cbf86757998a2339a0437208e=1723100816; Hm_lpvt_c99546cf032aaa5a679230de9a95c7db=1723100822"
}

data_list = []
for i in range(1, 6):
    url = f"https://match.yuanrenxue.com/api/match/3?page={i}"
    session.post('https://match.yuanrenxue.cn/jssm')
    for i in session.get(url).json()["data"]:
        data_list.append(i["value"])

from collections import Counter
counter = Counter(data_list)
most_common_element = counter.most_common(1)
(element, count) = most_common_element[0]
print(element)

```

