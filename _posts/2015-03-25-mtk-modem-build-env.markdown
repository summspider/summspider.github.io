---
layout: post
category: "mtk开发"
title:  "[MTK][Modem] MTK Modem 编译环境要求"
tags: [mtk modem 编译]
summary: "mtk modem编译环境要求"
---
###MTK官方支持环境
从L版本开始，MTK modem 编译支持的环境有下面两个：
* ubuntu 10.04 + local gcc 4.4.3 + linaro arm-none-eabi-gcc 4.6.2
* ubuntu 12.04 + local gcc 4.6.3 + linaro arm-none-eabi-gcc 4.6.2

###Ubuntu 14.04 支持
经过测试发现ubuntu 14.04上面也可以build pass，测试环境为
* ubuntu 14.04 + local gcc 4.8.2 + linaro arm-none-eabi-gcc 4.6.2

###Toolchain获取
linaro arm-none-eabi-gcc 目前在linaro官方网站已经没有binary可以下载，下载source需要自己编译。当然如果有mtk license可以到mtk申请一包toolchain使用。
这里我有一包linaro arm-none-eabi-gcc toolchain放到了百度网盘可供使用。下载链接为：<http://pan.baidu.com/s/1qWzB6zi>
