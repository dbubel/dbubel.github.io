---
author: dbubel
pubDatetime: 2024-10-04T15:22:00Z
title: Adding Dependencies from GitHub in Zig
slug: zig-adding-github-dependency
featured: false
draft: false
tags:
  - zig
  - build-system
description: A guide on how to add a dependency to your Zig project from GitHub.
---

## Adding a Dependency from GitHub

Zig’s documentation can be somewhat limited, and I found myself spending more time than expected figuring out how to add a logging package to my project. The Zig version used in this example is `0.14.0-dev.1702+26d35cc11`.

### Zig Fetch

You can use the `zig fetch` command to add dependencies from GitHub. For example:

```bash
zig fetch --save https://github.com/karlseguin/log.zig/archive/refs/heads/master.tar.gz
```

The key part of the URL is `heads/master.tar.gz`, which specifies the branch or package archive you want to use. You can adjust this URL to target a different branch or release version of the package.

For instance, to pull a specific release, you would use:

```bash
zig fetch --save https://github.com/karlseguin/log.zig/archive/refs/tags/v1.1.1.tar.gz
```

After running the command, your `build.zig.zon` file will be updated. It should look something like this:

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

### Modifying `build.zig`

To integrate the logging dependency into your project, add the following to your `build.zig` file:

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

### Usage

After adding the dependency, you can now import and use the `logz` module in your application as shown below:

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

Finally, run your application using:

```bash
zig build run
```

This setup allows you to successfully integrate a GitHub-hosted Zig package into your project and use it in your application.
