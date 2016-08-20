---
layout: post
title:  "Render JSON, thư viện nào mới tốt?"
date:   2015-06-23
categories: [ruby]
---

Đối với những người đã từng xây dựng API server, chắc hẳn ai cũng đã từng hơn một lần đau đầu với việc lựa chọn thư viện JSON. Bài viết sau đây hi vọng sẽ cho bạn cái nhìn khách quan nhất về các thư viện phổ biến trong cộng đồng ruby hiện tại.

## Các tiêu chí đánh giá:

### Có monkey-patch hay không?

Điều này có lẽ không cần phải nói thêm nhiều, mọi vấn đề của monkey-patching đã được Solnic nói rất rõ trong [bài viết gần đây của anh](http://solnic.eu/2015/06/06/cutting-corners-or-why-rails-may-kill-ruby.html). Do đó, nếu bạn có ý tưởng làm một ứng dụng nghiêm túc (thay vì chỉ là một prototype nhanh) thì bạn nên tránh xa các thư viện sử dụng monkey-patch.

### Template Engine và Object

Đây thuần tuý là vấn đề sở thích. Một số người thích sử dụng template thay vì các object cho việc render vì điều này thể hiện rõ vai trò của chúng: JSON thực chất cũng tương tự như HTML trong mô hình MVC, chỉ là một dạng hiển thị của dữ liệu. Ngoài ra, cấu trúc của template (ví dụ jbuilder) được cho là dễ đọc hơn cấu trúc object.

Ngược lại, một số lập trình viên lại thích sử dụng object hơn, vì chúng giúp họ linh hoạt hơn trong các thao tác xử lý (ví dụ kế thừa, fallback version, ...). Ngoài ra sử dụng object thường nhanh hơn template (vì chúng không cần phải được đọc bởi parser).

### Partial - DRY code

Một tiêu chí quan trọng khác là việc hỗ trợ và render partial. Partial giúp lập trình viên hạn chế việc lặp lại những đoạn render json trùng nhau, qua đó giúp code ngắn lại và (có lẽ là) ít lỗi hơn. Tuy nhiên, một số thư viện không hỗ trợ tốt vấn đề này (như jbuilder) khiến cho tốc độ render JSON suy giảm đáng kể khi sử dụng nhiều partial.

### Hỗ trợ Oj native

[Oj](https://github.com/ohler55/oj) là JSON parser và marshaller có tốc độ tốt nhất hiện nay. Hầu hết các project muốn tăng tốc API server của mình đều sử dụng thư viện này thay cho engine gốc của Rails/Ruby. Thư viện hỗ trợ tốt Oj sẽ giảm đáng kể công sức của lập trình viên khi áp dụng thư viện của mình vào project của họ.

### Code dễ đọc

Hầu hết các thư viện phổ biến hiện nay đều có cú pháp khá đơn giản, do đó việc đọc một file hoặc template là rất dễ dàng. Tuy nhiên, vẫn có một số thư viện có vấn đề về điều này:
- rabl có cú pháp rất phức tạp, người mới làm quen sẽ phải mất vài ngày để có thể sử dụng chúng thật thành thạo.
- acts_as_api để template của mình ở trong model, thoạt nghe thì rất tiện lợi, nhưng có lẽ bạn sẽ nghĩ lại khi user model của mình có số dòng ~1000.

### Coupling với model

Thông thường thì JSON được render từ model, tuy nhiên đôi lúc các PORO object cũng cần được render thành JSON đấy. Nếu bạn đang sử dụng active_model_serializers hay acts_as_api  thì xin chúc mừng, bạn sẽ không thể sử dụng chúng để render các object này đâu :)

### Versioning

> I need versioning without copy and pasting huge swathes of code. If I want to make a query optimization in an API endpoint I shouldn’t need to browse through every version of the API, applying it to each file.

Josh đòi hỏi như vậy trong [bài viết về các giải pháp API của Ruby](http://joshsymonds.com/blog/2013/02/22/existing-rails-api-solutions-suck/). Và mình nghĩ đòi hỏi này hoàn toàn hợp lý.

### Tốc độ

Vấn đề này chắc mình không phải giải thích nhiều, đây là một trong những yêu cầu quan trọng nhất của bất kỳ một thư viện nào.

Mình đã viết một bài về chi tiết các kết quả benchmark này trong [một bài viết khác](http://kipalog.com/posts/So-sanh-toc-do-cac-thu-vien-render-JSON).

## Tổng kết

|                           | [to_json](http://apidock.com/rails/ActiveRecord/Serialization/to_json)                                              | [acts_as_api](https://github.com/fabrik42/acts_as_api)                                          | [jbuilder](https://github.com/rails/jbuilder)           | [rabl](https://github.com/nesquena/rabl)     | [active_model_serializers](https://github.com/rails-api/active_model_serializers) | [roar](https://github.com/apotonick/roar)     |
|---------------------------|------------------------------------------------------|------------------------------------------------------|--------------------|----------|--------------------------|----------|
| Monkey-patch activerecord | Có                                                   | Có                                                   | Không              | Không    | Không                    | Không    |
| Cách render               | Object                                               | Object                                               | Template           | Template | Object                   | Object   |
| Hỗ trợ partial            | Không                                                | Có                                                   | Có                 | Có       | Có                       | Có       |
| Hỗ trợ Oj                 | Không                                                | Không                                                | Có              | Có       | Không                    | Có       |
| Cú pháp                   | Ban đầu thì đơn giản, càng rắc rối nếu code càng lớn | Ban đầu thì đơn giản, càng rắc rối nếu code càng lớn | Đơn giản           | Phức tạp | Đơn giản                 | Đơn giản |
| Couple với Rails   | Không                                                | Couple với activerecord                                                   | Couple với actionview              | Không    | Couple với activerecord                       | Không    |
| Hỗ trợ phiên bản          | Không                                                | Không                                                | Có                 | Có       | Có                       | Có       |
| Tốc độ                    | Nhanh                                          | Nhanh                                          | Bình thường/Chậm | Chậm    | Chậm hoặc nhanh tuỳ trường hợp| Bình thường     |