---
layout: post
title: (メモ) Centos環境でのメールサービス操作
author: [tunghust]
categories: [technology]
tags: [tips]
comments: true
---

Author: [@tunghust](https://github.com/tunghust)

## ■メールサービスを確認 
`sudo alternatives --display mta`

## ■Postfixメールキュー確認と削除
`sudo postqueue -p`  
`sudo postsuper -d ALL`

## ■qmailメールキュー確認と削除
`sudo /var/qmail/bin/qmail-qstat`  
`sudo rm -f /var/qmail/queue/*/*/*`

## ■qmailメール内容確認
`sudo cat /var/qmail/queue/mess/*/* |  grep -5 -e Date: -e From: -e To: -e Subject: -e Content -e =? | nkf -w`

