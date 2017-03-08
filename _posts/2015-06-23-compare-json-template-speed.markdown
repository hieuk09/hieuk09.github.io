---
layout: post
title:  "So sánh tốc độ các thư viện render JSON"
date:   2015-06-23
categories: ruby
---

Như đã nói trong [bài viết trước](http://kipalog.com/posts/Render-JSON--thu-vien-nao-moi-tot), mình sẽ chia sẻ tổng hợp kết quả của các benchmark mình đã dùng để so sánh tốc độ render JSON của các thư viện phổ biến hiện nay.

## Dữ liệu dùng để render

Các thử nghiệm của mình dựa nhiều trên ý tưởng của [bài so sánh RABL và AMS của THUVA THARMA](http://techblog.thescore.com/benchmarking-json-generation-in-ruby/). Mình cũng chia thành các trường hợp xử lý JSON tương tự:

### Ultra Simple

Đây là dữ liệu JSON đơn giản nhất, là dữ liệu của một cuốn sách, không có nested object và dữ liệu phức tạp nào:

```json
{
  "name": "Voluptatum repudiandae ad qui et sit aspernatur a et.",
  "isbn": "194307513-1",
  "genre": "asperiores"
}
```

### Simple

Đây là dữ liệu JSON tương đối đơn giản, dữ liệu cuốn sách ban đầu giờ có thêm thông tin tác giả:

```json
{
  "name":"A consequuntur quas saepe error est ex rerum eum.",
  "genre":"asperiores",
  "isbn":"236674115-4",
  "author": {
    "name":"Dr. Abigayle West",
    "birthday":"1964-02-23",
    "info":"Ut id molestiae. Nam suscipit recusandae. Nesciunt tempora numquam illum rerum debitis."
  }
}
```

### Complex

Đây là dữ liệu phức tạp nhất, có lẽ thường được sử dụng trong thực tế, bao gồm thông tin tác giả và các cuốn sách liên quan. Những cuốn sách này cũng có thông tin tác giả của chúng:

```json
{
  "name":"A consequuntur quas saepe error est ex rerum eum.",
  "genre":"asperiores",
  "isbn":"236674115-4",
  "author": {
    "name":"Dr. Abigayle West",
    "birthday":"1964-02-23",
    "info":"Ut id molestiae. Nam suscipit recusandae. Nesciunt tempora numquam illum rerum debitis."
  },
  "related_books": [
    {
      "name":"A consequuntur quas saepe error est ex rerum eum.",
      "genre":"asperiores",
      "isbn":"236674115-4",
      "author": {
        "name":"Dr. Abigayle West",
        "birthday":"1964-02-23",
        "info":"Ut id molestiae. Nam suscipit recusandae. Nesciunt tempora numquam illum rerum debitis."
      }
    },
    {
      "name":"Aperiam magni tempore veniam explicabo pariatur vel.",
      "genre":"asperiores",
      "isbn":"025870819-0",
      "author": {
        "name":"Alda O'Hara",
        "birthday":"1967-05-01",
        "info":"Sit sunt quibusdam natus qui est. Aut omnis maiores facilis est quibusdam. Cumque voluptatem sed qui consequatur autem."
      }
    },
    {
      "name":"Ullam aspernatur eius ipsam.",
      "genre":"asperiores",
      "isbn":"368983854-1",
      "author": {
        "name":"Madyson Schmeler II",
        "birthday":"1993-09-13",
        "info":"Dolor iste nobis eaque rerum similique vitae. Quas ea sed qui quos numquam. Quia dicta et et est hic ad."
      }
    }
  ]
}
```

### Collection

Ngoài việc benchmark một record như trên, với từng trường hợp, mình cũng benchmark việc render một mảng 50 record tương tự.

## Môi trường render

Các benchmark từ trước tới giờ mình thấy trên mạng chỉ so sánh tốc độ của các thư viện trong môi trường riêng biệt với nhau. Ở đây mình có thử nghiệm thêm với trường hợp render view thông qua Rails controller do đôi khi các thư viện này cũng ảnh hưởng đến tốc độ mà Rails render view.

Trong thử nghiệm render JSON ở model, do Jbuilder không thể sử dụng các template của views để render nên mình bỏ qua thư viện này.

## Implementation

Để đạt được tốc độ tối đa cho từng thư viện mình đã sử dụng một số điều kiện sau:
- render bằng gem Oj thay vì gem JSON mặc định của Rails
- với RABL, sử dụng options cache_sources là true
- với ROAR, sử dụng decorator thay vì extend
- với JBuilder, mình sử dụng 2 implementation: sử dụng partial và không sử dụng partial

Điều kiện benchmark:

- Sử dụng thư viện [benchmark-ips](https://github.com/evanphx/benchmark-ips) và [rspec](https://github.com/rspec/rspec).
- Cấu hình máy:
  + Macbook Air cài OS X 10.9.5
  + CPU: Core i7 1.7Ghz
  + RAM: 8GB 1600 MHz DDR3

Bạn có thể tham khảo source code tại [đây](https://github.com/hieuk09/benchmark_json_renderer).

## Kết quả

*Trong biểu đồ: đơn vị là iteration/s, càng lớn thì thư viện càng nhanh

### Trong model

#### Single record ultra simple

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/single_record_ultra_simple.png_2x2unpujlt)

#### Single record simple

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/single_record_simple.png_xs2wlp1fzw)

#### Single record complex

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/single_record_complex.png_oaatqk3y3d)

#### Collection ultra simple

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/collection_ultra_simple.png_srb5sn8qow)

#### Collection simple

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/collection_simple.png_86x89kaulw)

#### Collection complex

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/collection_complex.png_sg3ol8cp0i)

### Trong view

#### Single record ultra simple

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/view_single_record_ultra_simple.png_5xcjza1fi)

#### Single record simple

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/view_single_record_simple.png_qo98iqwf3d)

#### Single record complex

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/view_single_record_complex.png_qjfizhlv2n)

#### Collection ultra simple

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/view_collection_ultra_simple.png_ded83mqkh6)

#### Collection simple

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/view_collection_simple.png_42x4573ur7)

#### Collection complex

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/view_collection_complex.png_s8drp0835v)

## Nhận xét

Có thể thấy rằng tốc độ render thành JSON trong model của `to_json` là vượt trội so với việc sử dụng các thư viện khác, trong khi `rabl` quá chậm. Cụ thể khi render một record ultra simple, tốc độ của `to_json` nhanh gấp 48 lần so với thư viện chậm nhất (`rabl`) . Còn khi render 50 record complex, chênh lệch này còn lớn hơn, lên đến 60 lần.  Trong các thư viện còn lại, `acts_as_api` xử lý rất nhanh với một record, nhanh hơn nhiều so với `AMS` hay `ROAR`. Khi render collection, với cấu trúc simple và ultra simple, `acts_as_api` vẫn rất nhanh, nhưng lại tụt lại so với `ROAR` khi render cấu trúc complex.

Khi render trong view, có vẻ như thời gian xử lý trong `Rack` chiếm khá lớn thời gian nên chênh lệch không còn quá lớn như khi render JSON isolated. Tuy nhiên `to_json` và `acts_as_api` vẫn chiếm các vị trí dẫn đầu trong các benchmark. Chỉ có hai trường hợp:
- khi render collection ultra simple, thì tốc độ của `to_json` là chậm nhất, trong khi `AMS` hoàn thành bài test với nhiều iteration nhất.
- khi render collection simple, tốc độ của `ROAR` là nhanh nhất.

Trong tất cả các benchmark trong views với một record, `AMS` tỏ ra là thư viện chậm nhất, trong khi `rabl` chậm nhất khi render collection.

## Kết luận

Qua benchmark trên, ta có thể thấy các thư viện sử dụng template để render thường chậm hơn các thư viện sử dụng object. Ngoài ra, trong hầu hết trường hợp, sử dụng `to_json` thuần tuý vẫn là nhanh nhất, tuy nhiên bạn sẽ bỏ qua nhiều tính năng tiện lợi mà các thư viện khác mang lại. Đôi khi, có [nhiều vấn đề để bạn quan tâm hơn là tốc độ](http://kipalog.com/posts/Render-JSON--thu-vien-nao-moi-tot).

Hi vọng hai bài viết vừa qua sẽ giúp bạn có một cái nhìn tổng quan nhất để có thể lựa chọn thư viện phù hợp nhất cho project của mình.
