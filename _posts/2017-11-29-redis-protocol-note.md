---
layout: post
title: "Note about Redis protocol specification"
date:   2017-11-29
categories: programming note
---

This is the note that I create when reading about [Redis protocol
specification (RESP)](https://redis.io/topics/protocol).

## Networking Layer

RESP uses TCP, with request-response style.

## Description

- Has 5 types: Simple String, Errors, Integer, Bulk Strings and Array
- Every data end with `"\r\n"`
- Simple String starts with a “+”. The string include “\r\n”. Client will return the string.
- Error starts with a "-". Convention “-<error_type> <error_message>”. Client will raise and error.
- Integer starts with a “:”. Integer will be a signed 64-bit integer.
- Bulk Strings: starts with a “$”, followed by the string length, terminated by “\r\n”,
  followed by the actual string. The string is at most 512MB. Null String is “$-1\r\n”.
- Array: starts with a "\*",
  followed by the number of elements and “\r\n”, followed by the Type for each element.
  Array can contain mixed types. There is also Null Array “\*-1\r\r”
