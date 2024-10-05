---
author: dbubel
pubDatetime: 2024-10-05T15:22:00Z
title: Stream read file
slug: read-file-streaming
featured: false
draft: false
tags:
  - zig
  - streaming
  - file
  - arraylist
description: Simple example of reading a file into an ArrayList and fixed size buffer using Zig
---

## Stream a file into an ArrayList

Coming from a language like Go you might be spoiled by its `Reader` interfaces. The reader/writer concept does exist in Zig but is obviously in its pre `1.0.0` release state. This was tested with `0.14.0-dev.1702+26d35cc11`. The below shows how to read a file in zig in a performant mannor. Using an `ArrayList` and also a fixed size buffer.

## Buffered reader with `ArrayList` backing

```zig
const std = @import("std");
const print = std.debug.print;

pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    const allocator = gpa.allocator();

    const file = try std.fs.cwd().openFile("loadtest.js", .{});
    defer file.close();

    var buf_reader = std.io.bufferedReader(file.reader());
    const reader = buf_reader.reader();

    var line = std.ArrayList(u8).init(allocator);
    defer line.deinit();

    const writer = line.writer();
    while (true) {
        defer line.clearRetainingCapacity();
        reader.streamUntilDelimiter(writer, '\n', null) catch |err| {
            switch (err) {
                error.EndOfStream => {
                    return;
                },
                else => {
                    std.debug.print("error: {any}", .{err});
                    return;
                },
            }
        };
        print("{s}\n", .{line.items});
    }
}
```

```bash
./readfile_streaming_arraylist_alloc  2.61s user 0.28s system 99% cpu 2.891 total
```

## Buffered reader with a fixed size `[512]u8` buffer

```zig
const std = @import("std");
const print = std.debug.print;
pub fn main() !void {
    const file = try std.fs.cwd().openFile("loadtest.js", .{});
    defer file.close();

    var buf_reader = std.io.bufferedReader(file.reader());
    const reader = buf_reader.reader();

    var line: [512]u8 = undefined; // make sure your line len is less than 512
    var writer = std.io.fixedBufferStream(&line);

    while (true) {
        defer writer.reset();
        reader.streamUntilDelimiter(writer.writer(), '\n', null) catch |err| {
            switch (err) {
                error.EndOfStream => {
                    return;
                },
                else => {
                    std.debug.print("{any}", .{err});
                },
            }
        };

        print("{s}\n", .{line[0..writer.pos]});
    }
}

```

```bash
./readfile_streaming_buf  2.03s user 0.28s system 99% cpu 2.325 total
```

## Conclusion

As you can see there is a slight gain in performance with using a fixed size buffer. The file i used to test was 1.1gb in size. 
