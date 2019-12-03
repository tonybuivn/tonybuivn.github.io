---
layout: post
title: (Memo) Export & Import putty settings
author: [tunghust]
categories: [technology]
tags: [tips]
comments: true
---

Author: [@tunghust](https://github.com/tunghust)

Memo dành cho việc export & import putty settings sang 1 máy PC khác.  
※Môi trường: Windows

## Cách Export  
1. Khởi động `cmd` command `Widnows + R`  
2. Export sessions only  
`regedit /e "%USERPROFILE%\Desktop\putty-sessions.reg" HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions`
3. Export sessions with setting  
`regedit /e "%USERPROFILE%\Desktop\putty.reg" HKEY_CURRENT_USER\Software\SimonTatham`

## Các cách import
1. Double click
2. Dùng `cmd` command  
`regedit /i putty-sessions.reg`  
`regedit /i putty.reg`

※Chú ý:  
`SimonTatham` không phải là username.

