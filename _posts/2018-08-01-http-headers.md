---
layout: post
title: "TIL - HTTP headers: Content negotiation"
date:   2018-08-01
categories: programming
---

HTTP client and server can negotiate content type that is transported using
[header fields `Accept`, `Accept-Encoding`, `Accept-Language`, ...](https://en.wikipedia.org/wiki/Content_negotiation)

One of the usage of these fields is to compress the content to make it smaller,
thus faster to transport via the internet. There are several compress
algorithms, such as gzip and deflate. Some algorithms are supported in most
servers/clients, some are not (Android does not support deflate algorith).

In Ruby, some HTTP clients, such as [RestClient](https://github.com/rest-client/rest-client),
automatically add these headers to speed up the request/response time. Some clients,
such as [httprb](https://github.com/httprb/http), supports these headers, but
requires manual handling:

```ruby
# :auto_inflate is for automatically decompressing data
# `Accept-Encoding` header is used for signalling compressing in server
HTTP.use(:auto_inflate).headers('Accept-Encoding' => 'gzip, deflate').get(url)
```

You can compare the `content-length` header in the response with the below
command:

```ruby
# This usually have bigger content-length than the above command
HTTP.get(url)
```
