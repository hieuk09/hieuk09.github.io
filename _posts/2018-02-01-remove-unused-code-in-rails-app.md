---
layout: post
title: "Xoá code thừa trong rails app"
date:   2018-02-01
categories: ruby
---

Sau một thời gian phát triển thì Rails app thường phình lên khá to, và nếu không được dọn dẹp thường xuyên thì sẽ xảy ra tình trạng có nhiều code dư thừa, hoặc không được dùng đến nữa. Những đoạn code này, dù không chạy, nhưng vẫn gây ra vấn đề cho bạn:

- Tốn thời gian chạy test
- Tốn thời gian maintain (nếu bạn không biết là nó không còn được dùng nữa)
- Tốn thời gian parse và tìm kiếm của ruby khi hoạt động

Dưới đây là một số tool mình thu nhặt được để dọn dẹp bớt đống code dư thừa đó.

## Xoá method không dùng đến bằng [unused](https://github.com/joshuaclayton/unused)

[unused](https://github.com/joshuaclayton/unused) là một thư viện cực kỳ tuyệt vời, nó có thể giúp bạn tìm ra hầu hết các method và class hầu như không được sử dụng trong hệ thống và xoá chúng đi (bước này thì bạn phải tự làm). Cách sử dụng unused khá dễ:

- Cài đặt Ctags (MacOS mặc định có nhưng bạn vẫn phải install và ghi đè lên)

```shell
# MacOS
brew install ctags

# Ubuntu
apt-get install ctags
```

- Cài đặt unused:

```shell
# MacOS
brew tap joshuaclayton/formulae
brew install unused

# Other: bạn phải cài đặt stack, 
# cũng như phải thay đổi một chút config để cài thành công
stack update
stack install unused
```

- Chạy unused

```shell
ctags -R . # tạo tags file
unused
```

## Xoá partial dư thừa:

Gem [discover-unused-partials](https://github.com/vinibaggio/discover-unused-partials) rất hữu ích trong việc giúp bạn tìm ra những partial không còn được dùng đến trong codebase của bạn. Cách sử dụng cũng rất dễ dàng:

```shell
gem install discover-unused-partials
discover-unused-partials .
```

## Xoá các step dư thừa trong cucumber

Cucumber có options `--dry-run -f stepdefs` rất hữu ích trong việc tìm ra những step không được dùng đến nữa:

```shell
cucumber --dry-run -f stepdefs | grep -B 1 "NOT MATCHED BY ANY STEPS"
```

## Xoá các file ảnh không được sử dụng:

Anh Kumar có một đoạn [script](https://blog.botreetechnologies.com/remove-unused-images-from-rails-application-6d68eba67d8e) rất tiện lợi để tự động xoá những file ảnh không được reference bởi code trong rails app của bạn. Tuy nhiên, nên cẩn thận kiểm tra lại trước khi commit lên master:

```ruby
images = Dir.glob('app/assets/images/**/*')
unused_images = []
images.each do |image|
  unless File.directory?(image)
    puts "Checking #{image}..."
    result = `grep -nr #{File.basename(image)}* app/ `
    if result.empty?
      unused_images << image
      `rm -rf #{image}`
    end
  end
end
```

## Kết

Chúc các bạn xoá code vui vẻ :smile: