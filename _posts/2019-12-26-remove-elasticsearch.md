---
layout: post
title: "Cleanly remove elasticsearch on MacOS with homebrew"
date:   2019-12-26
categories: elasticsearch
---

Install elasticsearch without removing old version can cause issues when
running. In order to prevent that, we can cleanly elasticsearch with:

```
brew uninstall elasticsearch
rm -rf /usr/local/etc/elasticsearch
rm -rf /usr/local/var/lib/elasticsearch
```
