---
layout: post
title: SSH Port Forwarding - Không chỉ đơn thuần là SSH
author: [tunghust]
categories: [technology]
tags: [ssh, trick]
comments: true
---

Author: [@tunghust](https://github.com/tunghust)

## Bối cảnh
Bạn muốn truy cập vào server `dhs_blog` trên cổng `1234` từ `local` trong khi server đó chỉ `accept` kết nối từ server trung gian tạm gọi là `dhs_entrance`  
Khi đó giải pháp của bạn sẽ là gì?

![Ví dụ](https://kvlj0a.bn.files.1drv.com/y4mbkYpWs24oFxM2XIgFPLJHvXZ2BvcbMG3NEQw4jP5l_jgpvQ3QdLEeiFa7fcd6fZpzs5-qz9W1rLoT5dlyQpJY8Uj7KnTtTy0X5KfN0SXozq51x_longnE6A-ECpG7mFtkb5wOW8OvjnQjufxkNqvWdyhz_BYHuAp9JRZAWrZ8l1ks-F9jABGLICxn9FZIF5eGwJVcbVozu9LZwXOsmXIjA?width=541&height=218&cropmode=none "Ví dụ")  
*Hình 1: Vấn đề gặp phải*

## Giải quyết 
Sử dụng SSH Port Forwarding

Ở máy `local`
```
ssh -L 6789:dhs_blog:1234 dhs_entrance
#L option = Local
```

Lúc này việc thực hiện kết nối từ `local` thông qua port `6789` sẽ xảy ra các action ngầm phía sau:
- Kết nối từ `local` tới `dhs_entrance` qua port `ssh 22`
- Từ `dhs_entrance` thực hiện kết nối đến `dhs_blog` thông qua port `1234`
![Step](https://mucpqq.bn.files.1drv.com/y4mxbjK1BS7aWjeAzJGg2npIl5bODh2_WCrOfFy0PBN8cpK65ERFeMeKw1SOsOt3MsEWt2Iuh8O67SOA759TvrM7ZO2LNiwKEI5oy4_6alvTH70z9xo77saVyuGMlTnJ8PxuSTJe4Tgdz2j0Uza3AKcVwoiqQxWGWngFVzLlrdsYpMbweCgNuTvtNfQS-i8Vdhy75Y8CSrFLegZASGmEgWHCg?width=541&height=178&cropmode=none "Step")  
*Hình 2: Thực tế các bước xử lí bên trong*

## Ứng dụng phổ biến
- Được dùng trong `putty` chính là `SSH tunnel`
- Tool kết nối `DB Workbench`
- Tạo kết nối trực tiếp tới `DB Server` từ `local`
- Tạo kết nối trực tiếp tới `FTP Server` từ `local`
- etc...
