---
author: dbubel
pubDatetime: 2024-10-04T15:22:00Z
title: Adding Dependencies from GitHub in Zig
slug: zig-adding-github-dependency
featured: false
draft: true 
tags:
  - zig
  - http
  - server
  - thread
  - multithreaded
description: A simple http server using the Zig standard library to handle concurrent connections
---
## Zig `std.http.Server`
The Zig standard library is under active development. This example is using `0.14.0-dev.1702+26d35cc11` but the API is subject to constant change. 

The example below implements a very light wrapper around the `std.http` package implementing `init` and `run` functions.


