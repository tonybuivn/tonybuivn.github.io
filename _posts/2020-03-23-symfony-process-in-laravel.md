---
layout: post
title: Run command by Process in Laravel
author: [hungpq]
categories: [technology]
tags: [laravel, php]
comments: true
---

Author: [@qhung](https://github.com/qhung)

# Mở đầu
Chắc hẳn nhiều anh em đã và đang code dự án sử dụng Laravel Framework đã gặp phải những bài toán như là: Chạy 1 câu lệnh hay batch từ controller hay từ 1 batch khác.
Trong PHP thuần trước đây, câu lệnh exec hay được sử dụng tuy nhiên ở Laravel cung cấp 1 giải pháp khác cho bài toán này đó là `Process`.
Process đươc phát triển phía `Symfony Framework` và được Laravel kế thừa ở hiện tại.

# Symfony/Process
Đối với Laravel thì đã được tích hợp sẵn khi bạn download project về nên bạn không cần quan tâm đến việc cài đặt.
```text
https://symfony.com/doc/current/components/process.html
```

# Cú pháp/ cách dùng
Cơ bản, Process hỗ trợ cả đồng bộ và bất đồng bộ
## Xử lý đồng bộ
1 xử lý đồng bộ đơn giản như sau

```php
use Symfony\Component\Process\Exception\ProcessFailedException;
use Symfony\Component\Process\Process;

class TestBatch
{
    public function handler()
    {
        $process = new Process(['ls', '-la']);
        $process->run();
        
        // executes after the command finishes
        if (!$process->isSuccessful()) {
            throw new ProcessFailedException($process);
        }

        echo $process->getOutput();
    }
}
```

Hoặc để khởi tạo 1 Process chạy artisan
 ```php
$process = new Process(['php', 'artisan', 'batch:sample']);
```

## Xử lý bất đồng bộ
Với 1 process, gọi hàm `run` thì sẽ xử lý đồng bộ, còn dùng `start` sẽ được xử lý bất đồng bộ.
Ví dụ đơn giản như sau:
```php
    public function handler()
    {
        $process = new Process(['php', base_path('artisan'), 'batch:sample']);
        $process->start();

        while ($process->isRunning()) {
            echo 'Processing...'. "\n";
            sleep(5);
        }

        if ($process->isSuccessful()) {
            // some code
        }
    }
``` 

Với `isRunning()` ta có thể check được trạng thái đang chạy để xử lý phù hợp.

# Kết luận
Trên đây là 2 ví dụ đơn giản về sử dụng `Process` rất hữu dụng để xử lý bất đồng bộ trong ứng dụng web.
Các bạn có thể xem cụ thể hơn ở document của Symfony.
```
https://symfony.com/doc/current/components/process.html
```
