---
layout: post
title: Tìm hiểu về iNode trong Linux
author: [hungpq]
categories: [technology]
tags: [linux]
comments: true
---

Author: [@qhung](https://github.com/qhung)

Trong thời gian vận hành 1 service của công ty, một lần mình đã gặp phải lỗi bắn ra từ apache như sau
 ```
 No space left on device
 ```

Sau khi check df -h thì disk vẫn trống khoảng 30%, tuy nhiên khi check bằng command
```
bitnami@ip-172-26-6-241:~$ df -i
Filesystem      Inodes  IUsed   IFree IUse% Mounted on
udev             60319    322   59997    1% /dev
tmpfs            62293    435   61858    1% /run
/dev/xvda1     2560000 163399 2396601    7% /
tmpfs            62293      1   62292    1% /dev/shm
tmpfs            62293      3   62290    1% /run/lock
tmpfs            62293     16   62277    1% /sys/fs/cgroup
/dev/loop1          15     15       0  100% /snap/amazon-ssm-agent/1480
/dev/loop3          15     15       0  100% /snap/amazon-ssm-agent/1566
/dev/loop4       12852  12852       0  100% /snap/core/8935
tmpfs            62293      4   62289    1% /run/user/1000
/dev/loop0       12857  12857       0  100% /snap/core/9066

```
thì Iuse% của /dev/xvda1 là 100%

# iNode là gì?

Trong các hệ thống lưu  file system ext2, ext3, ext4 iNode có vai trò quản lý file, folder.
Các thuộc tính như create user, create datetime, group... đều là dữ liệu inode.

# iNode Number

Các file, folder đều được phân chia một iNode number. 
- iNode Number không trùng lặp.
- iNode Number có số lượng phù thuộc vào kích thước disk. (Nên mới xảy trường hợp no space như trên mình nói).


# Các thuộc tính được quản lý bởi iNode
- iNode Number
- UID
- GID
- Permission
- File Size
- File created datetime, modified datetime
- Vị trí vật lý của file nằm trên disk

※Tuy nhiên iNode không quản lý tên file.

# Kiểm tra inode number của file, folder

Để quản lý tình trạng sử dụng iNode hiện tại
```bash
df -i
Filesystem      Inodes  IUsed   IFree IUse% Mounted on
udev             60319    322   59997    1% /dev
tmpfs            62293    435   61858    1% /run
/dev/xvda1     2560000 163399 2396601    7% /
tmpfs            62293      1   62292    1% /dev/shm
tmpfs            62293      3   62290    1% /run/lock
tmpfs            62293     16   62277    1% /sys/fs/cgroup
/dev/loop1          15     15       0  100% /snap/amazon-ssm-agent/1480
/dev/loop3          15     15       0  100% /snap/amazon-ssm-agent/1566
/dev/loop4       12852  12852       0  100% /snap/core/8935
tmpfs            62293      4   62289    1% /run/user/1000
/dev/loop0       12857  12857       0  100% /snap/core/9066
```

Nhìn vào dữ liệu chúng ta có thể thấy như sau.

| Column | Giải thích |
| :--- | :---: |
| Inodes | Trên disk hiện tại, max iNode có thể tạo |
| IUsed | Số lượng iNode đã dùng |
| IFree | Số lượng iNode free hiện tại |
| IUse% | % iNode đã dùng |

Ở con server của mình check hiện tại, đối với device /dev/xvda1 thì

- 2560000 là giới hạn của iNode
- Đã sử dụng 163399 để lưu file/folder
- Còn lại 2396601 iNode
- % sử dụng là 7%.

Thử tạo 1 file mới.
```bash
cd /tmp/
touch new_file.txt

```

Check iNode number
```bash
ls -li new_file.txt
2084 -rw-rw-r-- 1 bitnami bitnami 0 Apr 29 23:01 new_file.txt # <- 2084

```

Check lại tình trạng iNode của device
```bash
df -i /dev/xvda1

Filesystem      Inodes  IUsed   IFree IUse% Mounted on
/dev/xvda1     2560000 163400 2396600    7% /
```

Ta thấy IUsed đã tăng từ 163399 -> 163400.

# Kết
Trong thực tế có thể bạn sẽ ít gặp những lỗi về iNode, tuy nhiên với các device có size nhỏ, đồng thời cũng lưu nhiều 
file/folder nhỏ thì lỗi này cũng có thể sẽ xảy ra. Lúc đó bạn có thể add thêm device hoặc xoá những file không cần thiết 
đi để giải phóng các free iNode./ 