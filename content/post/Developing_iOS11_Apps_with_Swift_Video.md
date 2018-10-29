---
title:  Developing iOS 11 Apps with Swift视频下载
slug: developing_ios11_apps_with_swift
date: 2018-10-29T16:15:15+08:00
description:
categories:
  - iOS开发入门
tags:
  - iOS
---


从Mac版的iTunes里找到课程，通过分享链接，获取下面的地址，
其实就是一个xml文件

[iTunes podcasts 视频解析地址](http://podcasts.apple.com/stanford/developing_ios11_apps.xml)

解析出所有的下载地址

```python
import xml.etree.ElementTree as ET
import requests
root = ET.fromstring(requests.get('http://podcasts.apple.com/stanford/developing_ios11_apps.xml').text)
for enclosure in root.findall('.//enclosure'):
    print(enclosure.get('url'))
```

<!--more-->

## 解析结果

### 视频下载地址(视频里内置了英文字幕，要播放器支持才能看到。也可以用字幕提取工具[CCExtractor](https://www.ccextractor.org/)提取字幕)

* <http://podcasts.apple.com/stanford/media/1_Introduction_to_iOS_11_Xcode_9_and_Swift_4_311-6554896743492737986-01_9_25_17_1080p_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/2_MVC_309-2503760600607007728-02_9_27_17_CS193p_720p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/504-8815564797834236275-Friday_01_9_29_17_WIP02_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/3_Swift_Programming_Language_319-2557083044702627203-03_10_02_17_prores_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/4_More_Swift_336-3848977901446740876-04_10_04_17_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/5_Drawing_317-8878116075149346380-05_10_09_17_prores_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/6_Multitouch_321-6728107643679048758-06_10_11_17_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/309-96054835436878188-07_10_16_17_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/327-7793456677118526749-08_10_18_17_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/533-6252825471341223932-Friday_02_WIP02_1080p_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/335-8360806109958293556-09_10_23_17_prores_CS193p_1080p_3mb_cc2.m4v>
* <http://podcasts.apple.com/stanford/media/311-695518005627000002-10_10_25_17_prores_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/531-9139141023410630962-Friday_03_10_27_17_WIP02_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/335-5035465745252410791-11_10_30_17_prores_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/303-5025198319972677062-12_11_01_17_prores_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/312-9097236878835223246-13_11_06_17_prores_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/305-1366423083051400424-14_11_08_17_prores_1_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/532-3346936006580998385-15_11_13_17_prores_CS193p_1080p_3mb_cc.m4v>
* <http://podcasts.apple.com/stanford/media/511-8983128522622077657-16_11_15_17_prores_CS193p_1080p_3mb.m4v>
* <http://podcasts.apple.com/stanford/media/506-4176003801724267012-17_11_29_17_WIP02_CS193p_1080p_3mb_cc.m4v>

### PDF课件下载地址

* <http://podcasts.apple.com/stanford/media/Lecture_1_Slides.pdf>
* <http://podcasts.apple.com/stanford/media/Reading_1_Intro_to_Swift.pdf>
* <http://podcasts.apple.com/stanford/media/Lecture_2_Slides.pdf>
* <http://podcasts.apple.com/stanford/Programming_Project_1_Concentration.pdf>
* <http://podcasts.apple.com/stanford/media/Lecture_3_Slides.pdf>
* <http://podcasts.apple.com/stanford/media/Reading_2_Intro_to_Swift.pdf>
* <http://podcasts.apple.com/stanford/media/Lecture_4_Slides.pdf>
* <http://podcasts.apple.com/stanford/Programming_Project%202_Set.pdf>
* <http://podcasts.apple.com/stanford/media/Lecture_5_Slides.pdf>
* <http://podcasts.apple.com/stanford/media/336-3367719085568248138-CS193P_F17_Reading_3.pdf>
* <http://podcasts.apple.com/stanford/media/Lecture_6_Slides.pdf>
* <http://podcasts.apple.com/stanford/media/309-1475895300459033080-CS193P_F17_Assignment_3.pdf>
* <http://podcasts.apple.com/stanford/media/320-5503734535583450465-CS193P_F17_Lecture_7.pdf>
* <http://podcasts.apple.com/stanford/media/335-3416648937156657194-CS193P_F17_Lecture_8.pdf>
* <http://podcasts.apple.com/stanford/media/309-2441628197055099936-CS193P_F17_Assignment_4.pdf>
* <http://podcasts.apple.com/stanford/media/332-3154341613995510587-CS193P_F17_Lecture_9.pdf>
* <http://podcasts.apple.com/stanford/media/322-6515997048783028736-CS193P_F17_Lecture_10.pdf>
* <http://podcasts.apple.com/stanford/media/317-8823617114003497609-CS193P_F17_Lecture_11.pdf>
* <http://podcasts.apple.com/stanford/media/306-4082649130340532415-CS193P_F17_Lecture_12.pdf>
* <http://podcasts.apple.com/stanford/media/308-8518403617003934463-CS193P_F17_Assignment_5.pdf>
* <http://podcasts.apple.com/stanford/media/323-4247208383177102782-CS193P_F17_Lecture_13.pdf>
* <http://podcasts.apple.com/stanford/media/322-5794459892624250293-CS193P_F17_Lecture_14.pdf>
* <http://podcasts.apple.com/stanford/media/307-9138746427331321981-CS193P_F17_Assignment_6.pdf>
* <http://podcasts.apple.com/stanford/media/522-830176750758320683-CS193P_F17_Lecture_15.pdf>
* <http://podcasts.apple.com/stanford/media/512-802352814004650525-CS193P_F17_Lecture_16.pdf>
* <http://podcasts.apple.com/stanford/media/520-5280642336803569160-CS193P_F17_Lecture_17_Slides.pdf>


## 中文字幕

网上找到的中文字幕

<https://github.com/ApolloZhu/Developing-iOS-11-Apps-with-Swift>




