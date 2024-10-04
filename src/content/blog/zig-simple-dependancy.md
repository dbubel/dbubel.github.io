---
author: dbubel
pubDatetime: 2024-09-29
title: Adding dependencies from github
slug: zig-adding-github-dependency
featured: false 
draft: false 
tags:
    - zig
    - dependancy
description:
Adding a dependency to your zig project from github
---

## Adding a dependency from Github

Zig documentation is lack luster at best. I spent far too long figuring out how to simply add a logging package to my toy applicaton. The verion of Zig that I did this with was `0.14.0-dev.1702+26d35cc11`.

## `zig fetch`

```bash
zig fetch --save https://github.com/karlseguin/log.zig/archive/refs/heads/master.tar.gz
```

The above command should modify your `build.zig.zon` to look similar to this

```zig
.{
    .name = "http",
    .version = "0.0.0",
    .dependencies = .{
        .logz = .{
            .url = "https://github.com/karlseguin/log.zig/archive/refs/heads/master.tar.gz",
            .hash = "12204b771f0e40f27c6218a730dbd610583a3d7350756ff7961ebcbfd47db28eecda",
        },
    },
    .paths = .{
        "build.zig",
        "build.zig.zon",
        "src",
    },
}

```
