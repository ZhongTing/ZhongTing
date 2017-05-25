---
title: Python3在Windows下的中文亂碼處理
date: 2016-03-21 21:27
category: python
tags: encoding
keywords: 
excerpt: MAC, Linux 系統預設編碼utf8，而Windows預設使用ANSI編碼，因此windows下執行的python script會遇上不少的編碼麻煩。
---



<!-- excerpt -->

筆者是個python新手，不過最近論文需要，所以用python寫了一支load json預處理的程式
因為json內容大部分是中文，耳聞Python3在中文處理這方面比Python2還要來的好
於是使用了Python3.5來撰寫程式
沒想到在window下，這是一個悲劇的開始XDD

首先這是一個接json的程式
```python
from urllib import request

req = request.urlopen(jsonUrl)
print(req.read())
```
但馬上會發現中文的地方會變成\x8e\x6e之類的亂碼
接著直覺性的會想要轉碼
因為req.read()出來的型態是byte
所以下了decode轉成unicode

```python
from urllib import request

req = request.urlopen(jsonUrl)
encoding = req.headers.get_content_charset()
print(req.read().decode(encoding))
```

然後就出現類似這樣的錯誤

> UnicodeEncodeError: 'cp950' codec can't encode character '\ufffd' in position 5564: illegal multibyte sequence

起初只是單純以為decode沒有處理error
不過即使改成req.read().decode(encoding, errors='ignore')也無法改善
後來才發現是python print會偵測終端機的編碼轉碼輸出
所以unicode轉window cmd預設編碼cp950如果遇到無法解析的字就會報這個錯誤
除了改cmd的編碼之外，另外一個解法就是先自己encode成cp950並忽略掉錯誤再轉回unicode...

```python
from urllib import request
import sys

req = request.urlopen(jsonUrl)
encoding = req.headers.get_content_charset()
data = req.read().decode(encoding).encode(sys.stdin.encoding, 'replace').decode(sys.stdin.encoding)
print(data)
```

到這一步驟已經可以成功在終端機顯示中文了
但如果你像筆者一樣使用sublime text3，會發現預設的build system沒辦法正常print出中文
原因是sublime預設的編碼是utf-8要特別設定build system並將編碼指定為cp950才可以

```json
{
    "cmd": ["python", "-u", "$file"],
    "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
    "selector": "source.python",
    "encoding" : "cp950"
}
```
用上面的設定就可以在sublime成功印出中文囉!!

