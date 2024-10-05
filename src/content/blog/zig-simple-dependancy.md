---
author: dbubel
pubDatetime: 2024-09-18T15:22:00Z
title: Adding dependencies from github
slug: zig-adding-github-dependency
featured: false
draft: false
tags:
  - zig
  - build-system
description: Adding a dependency to your zig project from github
---

## Adding a dependency from Github

Zig documentation is lack luster at best. I spent far too long figuring out how to simply add a logging package to my toy applicaton. The verion of Zig that I did this with was `0.14.0-dev.1702+26d35cc11`.

## Zig fetch

```bash
zig fetch --save https://github.com/karlseguin/log.zig/archive/refs/heads/master.tar.gz
```

The important thing to note here is the end of the url `heads/master.tar.gz` denotes which package archive that your package will be using. You can specify different paths based on which branch, or release that you want to include in your build.

So for example if you wanted to pull a specific release you would do

```bash
zig fetch --save https://github.com/karlseguin/archive/refs/tags/v1.1.1.tar.gz
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

## Build.zig

To add the dependency to your `build.zig` file you can do the following.

```zig
...
const logger = b.dependency("logz", .{ .target = target, .optimize = optimize });

const exe = b.addExecutable(.{
    .name = "http",
    .root_source_file = b.path("src/main.zig"),
    .target = target,
    .optimize = optimize,
});

exe.root_module.addImport("logz", logger.module("logz"));
...
```

## Usage

You should now be able to import the module in your application like this

```zig
const std = @import("std");
const log = @import("logz");

pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};

    try log.setup(gpa.allocator(), .{
        .level = .Info,
        .pool_size = 100,
        .buffer_size = 4096,
        .large_buffer_count = 8,
        .large_buffer_size = 16384,
        .output = .stdout,
        .encoding = .logfmt,
    });
    defer log.deinit();

    log.info().string("path", "hello").int("ms", 1).log();
...
```

And run your program via `zig build run`
