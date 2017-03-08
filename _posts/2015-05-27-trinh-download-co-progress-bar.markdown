---
layout: post
title:  "Viết trình download có thanh tiến trình với ruby"
date:   2015-05-27
categories: ruby
---

Khi làm việc với Ruby và Rails, có lẽ không ít lần các bạn đã gặp các tác vụ download file về server của mình. Ruby hỗ trợ nhiều công cụ download khác nhau, từ đơn giản đến phức tạp, như `Net::HTTP`, `OpenURI` hay `Mechanize`, ... Tuy nhiên, các thư viện này không có sẵn chức năng hiển thị tiến trình (progress) cho người dùng, do đó đôi lúc bạn sẽ không biết là chương trình của mình đang chạy hay là đã treo rồi.

Bạn có thể dùng kết hợp gem `ruby-progressbar` và `OpenURI` để lấy thông tin và hiển thị tiến trình download. Một thanh tiến trình để hiển thị có thể được khởi tạo đơn giản như sau:

```ruby
progress_bar = ProgressBar.create(
  starting_at: 0,
  total: nil,
  format: "%a %B %p%% %r KB/sec",
  rate_scale: lambda { |rate| rate / 1024 }
)
```

Sau khi lấy được header của file, bạn cần gán nó cho độ dài của thanh tiến trình. `OpenURI` hỗ trợ `content_length_proc` callback để làm điều này:

```ruby
content_length_proc = Proc.new { |content_length|
  progress_bar.total = content_length
}
```

Bạn cần cập nhật độ dài của thanh tiến trình mỗi khi dữ liệu được tải về. `OpenURI` cũng hỗ trợ `progress_proc` callback để giúp bạn làm điều này:

```ruby
progress_proc = Proc.new { |bytes_transferred|
  progress_bar.progress = bytes_transferred
}
```

Đôi khi, header của file sẽ không chính xác sẽ dẫn đến việc số byte tải về vượt quá độ dài của thanh tiến trình, khi đó `ruby-progressbar` sẽ trả về một exception. Bạn cũng cần xử lý trường hợp này:

```ruby
progress_proc = Proc.new { |bytes_transferred|
  if progress_bar.total && progress_bar.total < bytes_transferred
    progress_bar.total = nil
  end
  progress_bar.progress = bytes_transferred
}
```

Bạn có thể xem đầy đủ code của chương trình này trên [github](https://gist.github.com/hieuk09/09d485626911bdfd7c79). Như vậy là bạn đã có một trình download với thanh tiến trình rất đẹp rồi nhỉ:

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/Screen%20Shot%202015-04-19%20at%2010.39.00%20pm.png_aishpo1na)

Ngoài ra, bạn còn có thể tận dụng các phần mềm download hiện có trên hệ thống bằng việc sử dụng hàm `system`. Hầu hết các trình download này đều mặc định hiển thị thanh tiến trình khi download. Sau đây là implementation của một số trình download phổ biến.

### CURL
curl là trình download phổ biến nhất trên các hệ điều hành Unix-based. Được cài sẵn trên hầu hết các distro Linux và các phiên bản Mac. Bạn có thể dùng gem `Curb` hoặc như sau:

```ruby
system("curl -A 'Wget/1.8.1' --retry 10 --retry-delay 5 --retry-max-time 4  -L #{link} -o #{file_path}")
```

### WGET
wget hầu như luôn có trên các distro Linux, còn trên Mac thì bạn thường phải cài thêm.

```ruby
system("wget -c #{link} -O #{file_path}")
```

### ARIA2
aria2 là một trình download gọn nhẹ, có thể cài đặt đơn giản trên Linux hoặc Mac. Điểm mạnh của aria2 là sử dụng nhiều connection song song khiến cho tốc độ download nhanh gấp 2-3 lần `curl` hay `wget`.

```ruby
system("aria2c #{link} -U 'Wget/1.8.1' -x4 -d #{File.dirname(file_path)} -o #{File.basename(file_path)}")
```
