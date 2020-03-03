---
layout: post
title: Tản mạn về Redis. What and Why ?
author: [tony]
categories: [technology]
tags: [redis, cache, nosql]
comments: true
bigimg: /img/redis-cover.png
---

Author: [@tonybui](https://github.com/tonybuivn)

# 1. Mở đầu
Gần đây, dự án mình đang tham gia bắt đầu tăng số lượng user một cách nhanh chóng, dẫn đến việc tăng số lượng request đến AWS, tăng thời gian phản hồi, và điều này cũng phát sinh một lượng chi phí không hề nhỏ. Trước đây mình cứ nghĩ tối ưu data chỉ có thể được thực hiện thông qua việc tối ưu truy vấn database

... nhưng mình đã nhầm

Sau khi được chỉ giáo bởi một vài tiền bối kiến trúc sư hệ thống, mình quyết định sử dụng một thứ gọi là Cache. Vậy...

# 2. Cache là gì
Nói một cách đơn giản, Cache là một vùng nhớ lưu trữ dữ liệu tạm thời, các dữ liệu này được tái sử dụng trong tương lai nhằm tăng tốc độ truy vấn. Khi đó ứng dụng không cần request đến database server để lấy dữ liệu nữa mà sẽ lấy dữ liệu trực tiếp từ bộ nhớ đệm (Cache). 

Trong bài này, chúng ta sẽ sử dụng Redis làm bộ nhớ đệm để lưu trữ.

# 4. Redis là gì ?
Theo định nghĩa trên trang chủ của [Redis](https://redis.io/), thì Redis là một mã nguồn mở (theo BSD License) lưu trữ dữ liệu dạng cấu trúc trong bộ nhớ (in-memory), được sử dụng như một database, bộ đệm và message broker.

Redis hỗ trợ nhiều cấu trúc dữ liệu khác nhau như là Strings, Hashes, Lists, Sets,...

{: .box-note}
**Rails related:** Với những dự án sử dụng Rails phía server, Redis thường được sử dụng làm bộ nhớ đệm lưu trữ các Job từ Sidekiq dưới dạng Key - Value.

# 5. Vì sao chúng ta nên dùng Redis ?
1. Nó siêu nhanh. Có lẽ bởi vì nó được viết bằng C chăng :))
![Flash](https://media.giphy.com/media/3oriNYQX2lC6dfW2Ji/giphy.gif)

2. Là một NoSql Database. Nghe thật fancy phải không :))
![NoSQL](https://miro.medium.com/max/700/1*b9ST4A6cvmAcmddepxP2pg.gif)

3. Hiện tại Redis đang được sử dụng bởi các ông lớn trong làng công nghệ như Github, Weibo, Pinterest, Snapchat, Craigslist, Digg, Stackoverflow, Flickr,... Cũng có chút yên tâm đúng ko nào :v

4. Tiết kiệm số request đến cloud database của bạn, và hiển nhiên là cũng tiết kiệm một khoản tiền đáng kể cho bạn luôn.

5. Dễ sử dụng, thân thiện với dev. Redis hiện tại đang được support bởi hầu hết các ngôn ngữ lập trình (đây có lẽ là đặc quyền khi sử dụng một công nghệ mã nguồn mở). Những ngôn ngữ được hỗ trợ bao gồm JavaScript, Java, Go, C, C++, C#, Python, Objective-C, PHP và hầu hết các ngôn ngữ nổi tiếng khác.

6. Điều cuối cùng và có lẽ cũng là một điều hiển nhiên, Redis là open-source và rất ổn định. Vậy còn chần chờ gì mà không say "Yes" với Redis.

![super-cool](https://miro.medium.com/max/986/1*NaUZFuHTRIZY23qMsgfb4g.gif)

*Redis be like, "I'm super cool"*

# 6. Cài đặt và start Redis server
## 6.1 Cài đặt
Có nhiều cách để cài đặt Redis. Các bạn dùng Mac thì có thể cài đặt một cách đơn giản thông qua brew
```
$ brew install redis
```

Với các bạn không dùng Mac có thể tham khảo cách cài đặt khác trên trang chủ của [Redis](https://redis.io/topics/quickstart)

## 6.2 Thiết lập chế độ autostart cho Redis

・ Chạy Redis mỗi khi khởi động máy tính
```
$ ln -sfv /usr/local/opt/redis/*.plist ~/Library/LaunchAgents
```

・ Tắt chế độ autostart Redis khi khởi động máy tính
```
$ launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.redis.plist
```

## 6.3 Khởi động Redis server 

・ Sử dụng `configuration file` (mình hay dùng cách này)
```
$ redis-server /usr/local/etc/redis.conf
```

・ Thông qua “launchctl”.
```
$ launchctl load ~/Library/LaunchAgents/homebrew.mxcl.redis.plist
```

## 6.4 Check xem Redis đã hoạt động hay chưa
```
$ redis-cli ping
```
Nếu server trả về `PONG`, vậy là bạn đã hoàn thành việc cài đặt `Redis`

# 6. Một vài sample command của Redis
Okay lý thuyết như vậy là đủ rồi. Bây giờ chúng ta sẽ bắt tay vào vọc vạch Redis một chút nhé. Trong bài này mình sẽ sử dụng Redis-CLI để minh hoạ.

> Cho bạn nào chưa biết thì Redis-CLI (Redis command line interface) là giao diện dòng lệnh của Redis, là một chương trình đơn giản cho phép chúng ta gửi các mã lệnh đến Redis, và đọc các phản hồi trả về từ server trực tiếp thông qua terminal.

Để bắt đầu Redis-CLI, chúng ta dùng lệnh
```
$ redis-cli
```

Mặc định Redis-CLI chạy trên port `6379`

・ SET (Thiết lập một Key)
```
127.0.0.1:6379> SET foo "Hello DHS"
OK // setting a key
```

・ GET (Lấy ra một Key)
```
127.0.0.1:6379> GET foo
"Hello DHS" // getting a key
```

・ DEL (Xoá một Key)
```
127.0.0.1:6379> GET foo 
"Hello DHS" // getting a key
127.0.0.1:6379> DEL foo
(integer) 1 // key just got deleted
127.0.0.1:6379> GET foo
(nil) // since key is deleted therefore, result is nil.
```

・ SETEX (Thiết lập một Key có thời hạn)
```
127.0.0.1:6379> SETEX foo 40 "Hello DHS will be expired in 40 seconds"
OK // key has been set with 40 seconds as expiration
```

・ TTL (Total Time left - Tổng thời gian còn lại của một Key cho đến khi bị timeout)
```
127.0.0.1:6379> TTL foo
(integer) 36 // 36 seconds left to timeout
```

・ PERSIST (Xoá bỏ thời gian timeout khỏi Key, có nghĩa Key sẽ không bị expire nữa)
```
127.0.0.1:6379> PERSIST foo
(integer) 1 // turning the key from volatile to persistent (key won't expire)
```

・ RENAME (Đổi tên của một Key)
```
127.0.0.1:6379> RENAME foo bar
OK // renaming the key 'foo' as bar
```

・ FLUSHALL (Giải phóng các Key đã được lưu)
```
127.0.0.1:6379> flushall
OK // just got flushed
```

# 7. [Nâng cao] Dùng KEYS PATTERN để lấy ra các Key theo ý muốn
Chúng ta dùng lệnh `KEYS` để lấy ra danh sách các Key theo quy ước mà chúng ta muốn. Cái này gần giống như Regex (gần giống thôi nhé, bạn không thể áp dụng tất cả các KEYS pattern của Redis vào Regex trong các ngôn ngữ lập trình và ngược lại)

Về cơ bản chúng ta có các pattern như sau
- h?llo matches hello, hallo and hxllo
- h*llo matches hllo and heeeello
- h[ae]llo matches hello and hallo, but not hillo
- h[^e]llo matches hallo, hbllo, ... but not hello
- h[a-b]llo matches hallo and hbllo

Example

・ Thiết lập Key - Value
```
127.0.0.1:6379> MSET firstname Tony lastname Bui middlenames Something age 28 job engineer
OK
```

・ Trả về tất cả các Key
```
127.0.0.1:6379> KEYS *
1) "middlenames"
2) "job"
3) "lastname"
4) "firstname"
5) "age"
```

・ Trả về các Key có kết thúc bằng xâu `name`
```
127.0.0.1:6379> KEYS *name
1) "lastname"
2) "firstname"
```

・ Trả về các Key có chứa xâu `name`
```
127.0.0.1:6379> KEYS *name*
1) "middlenames"
2) "lastname"
3) "firstname"
```

・ Trả về Keys có bắt đầu bằng ký tự `a` và tiếp sau đó phải có 2 ký tự bất kỳ
```
127.0.0.1:6379> KEYS a??
1) "age"
```

## Tóm váy lại, Redis thật Cool phải không nào.
