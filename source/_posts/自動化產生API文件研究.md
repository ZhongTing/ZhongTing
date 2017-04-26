---
title: A Survey of Rest API document generator 自動化產生API文件之研究
date: 2016-08-16 17:20
category: survey
tags: restful api
---

API文件除了word, google doc之外，你可以有更多選擇
<!-- more -->
現在有許多開源社群能夠自動產生API文件
[API Blueprint](https://apiblueprint.org)是一種開源的API語言
主要使用markdown語法來描述你的API
目前有許多工具可以直接從source code parse出API描述檔案
或者你也可以使用editor直接撰寫API描述檔案
之後再把API描述檔案render成html靜態文件

除了有美美的文件以及能夠解決文件與程式碼不同步的問題之外
還有許多人做出實用的小工具
例如可以將API描述檔案轉承對應的source code(這是一個遠大的理想，目前實用性也許還不高...）
或者將API描述檔案轉成Postman的script
也有套件可以直接根據API描述檔案一鍵建立mock server
API bluerprint自成一種開源社群
你可以貢獻自己寫的任何工具！


註一
[Swagger](http://swagger.io)與[Raml](http://raml.org/index.html)也是相似於API Blueprint的社群

註二
有些付費的服務整合的還滿好的
[Apiary](https://apiary.io)，目前支援APIBlueprint與Swagger

註三
除了以上所提及的社群，也有一些專案可以產生API文件
[APIDoc](http://apidocjs.com), [Slate](https://github.com/lord/slate)