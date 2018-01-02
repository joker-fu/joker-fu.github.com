---
title: win10 小箭头去除与恢复
copyright: true
date: 2018-01-02 13:43:12
categories: tool
tags: tool
---

###### 1.去掉小箭头

```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Icons" /v 29 /d "%systemroot%\system32\imageres.dll,197" /t reg_sz /f
taskkill /f /im explorer.exe
attrib -s -r -h "%userprofile%\AppData\Local\iconcache.db"
del "%userprofile%\AppData\Local\iconcache.db" /f /q
start explorer
pause
```

复制上面的代码，新建一个txt文件，粘贴后另存为.bat文件，然后以管理员身份打开。

###### 2.恢复小箭头

```
reg delete "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Icons" /v 29 /f
taskkill /f /im explorer.exe
attrib -s -r -h "%userprofile%\AppData\Local\iconcache.db"
del "%userprofile%\AppData\Local\iconcache.db" /f /q
start explorer
pause
```
同理，复制上面的代码，新建一个txt文件，粘贴后另存为.bat文件，然后以管理员身份打开。
