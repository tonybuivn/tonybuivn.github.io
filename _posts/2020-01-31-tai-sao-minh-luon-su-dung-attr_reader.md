---
layout: post
title: Tại sao chúng ta nên sử dụng attr_reader khi truy cập vào instance variables
author: [Tony]
categories: [technology]
tags: [ruby, rails]
comments: true
---

Author: [@tonybui](https://github.com/tonybuivn)

Phương thức `attr_reader` của ruby có thể được sử dụng để tự động sinh ra một `getter` cho một instance variable cho trước.

Nói một cách đơn giản, nếu chúng ta viết

```ruby
attr_reader :engine
```

thì sẽ tương đương với
```ruby
def engine
  @engine
end
```

Với programming style này, mình luôn sử dụng `attr_reader` để access vào `instance variable` (bạn cũng có thể gọi là `getter`). Bài viết này mình sẽ giải thích lý do vì sao chúng ta nên sử dụng style đó.

Giả sử mình có một class `Car` như sau để mô tả các thuộc tính của một chiếc xe

```ruby
class Car

  attr_reader :owner
  attr_reader :license_plate

  private

  attr_reader :engine
  attr_reader :gearbox

  public

  def initialize(owner:, license_plate:)
    /# initialization code/
  end

  /# other methods/
end
```

Và dưới đây là các lý do cho việc lựa chọn cách viết này
### 1. Đặt tất cả các `attr_readers` ở trên cùng của class*
Điều này biến 1 class trở thành một dạng document, bạn mở 1 class và ngay lập tức có thể hiểu các thuộc tính bên trong của class là gì.

Hơn thế nữa, nếu class của bạn không có thuộc tính nào là `attr_writer` hoặc `attr_acesor`, bạn có thể nhanh chóng biết ngay class đó là mutable (các thuộc tính của class có thể thay đổi trong suốt 1 vòng đời - lifetime của nó) hoặc immutable (các thuộc tính của class không thể thay đổi sau khi khởi tạo - lưu ý rằng điều này chỉ đúng với bản thân object đó, không áp dụng cho các object được class đó tham chiếu đến, ví dụ như một biến trỏ đến một unfrozen array)

Bằng cách áp dụng rule đơn giản này, bạn có thể nắm bắt được rất nhiều thông tin hữu ích một cách nhanh chóng.

### 2. Sử dụng `attr_readers` để access vào instance variables từ bên trong 1 class
Điều này sẽ giúp chúng ta tránh bị type nhầm. Giả sử chúng ta thêm method sau đây vào class `Car` ở trên, nhưng chẳng may gõ sai (`@eNgine` thay vì `@engine`)
```ruby
def start
  @gearbox.neutral
  @eNgine.start
end
```

Điều gì sẽ xảy ra nếu chúng ta cố gắng chạy đoạn code trên

```
$ ruby car.rb 
car.rb:20:in `start': undefined method `start' for nil:NilClass (NoMethodError)
        from car.rb:24:in `<main>'
```

Tại sao `engine` của chúng ta lại bị `nil`. Nếu không biết trước là chúng ta có một lỗi đánh máy sai, thì có thể sẽ mất rất nhiều thời gian để bạn tìm ra được nguyên nhân thực sự đằng sau.

Do đó thay vì access trực tiếp vào các `instance variable`, hãy sử dụng method mà `attr_reader` đã tạo ra

```ruby
def start
  gearbox.neutral
  eNgine.start
end
```

Và điều gì sẽ xảy ra nếu chúng ta chạy đoạn code mới này

```
$ ruby car.rb 
car.rb:20:in `start': undefined local variable or method `eNgine' for #<Car:0x000000c9d9522d50> (NameError)
Did you mean?  @engine
        from car.rb:24:in `<main>'
```

Chúng ta có thể thấy output lỗi đã rõ ràng hơn rất nhiều đúng không : chưa có một `eNgine` nào được khai báo, và `Did you mean? @engine`. Bạn có thể ngay lập tức nhận ra mình đã gõ sai và sửa nó.

### 3. Những lợi ích của việc sử dụng một getter
Khi bạn sử dụng một method để access vào `instance variable`, nó trở nên dễ dàng hơn rất nhiều khi bạn muốn refactor nó sau này mà không cần phải update tất cả những nơi `instance variable` đó đã được sử dụng.

Ví dụ, giả sử chúng ta muốn khởi tạo theo kiểu lazy (hay có thể gọi là lưu cache của `instance variable`), chúng ta có thể sửa đoạn code sau

```ruby
attr_reader :engine
```

sang

```ruby
def engine
  @engine ||= EngineFactory.create
end
```

### 4. Thậm chí chúng ta cũng nên sử dụng cho private instance variables
Vì tất cả những ưu điểm trên đều có thể áp dụng cho các `instance variables` mà không mở rộng ra bên ngoài của 1 class, nên mình recommend các bạn sử dụng `attr_reader` cho các variables này, ngay cả khi bạn thường không làm như vậy.
