---
layout: post
title:  "resolv - Writing DNS resolver in Zig"
date:   2023-06-04 12:01:55 +0530
categories: update
---

_Updated on_: 2023-06-10

## My getting started with Zig

`All your codebase are belong to us.`

I got into learning zig recently after seeing lots of zig videos/streams and reading articles on zig.
I went through chapter 0 and 1 from [ziglearn.org](https://ziglearn.org), and went through all the available challenges from [ziglings](https://github.com/ratfactor/ziglings).

Even after going through them, i did not feel confident about my zig. So, insipired by [this twitter post by @b0rk](https://twitter.com/b0rk/status/1663960242667782184), I decided to write DNS resolver program in zig.

## resolv

[`resolv`](https://github.com/e-aakash/resolv/) is my try at implementing DNS resolver in zig. Instead of trying to follow b0rk's implementation/explaination of DNS resolver, I decided to try to read the spec myself, and try to implement it.

I went over [RFC 1035](https://datatracker.ietf.org/doc/html/rfc1035), and created small notes (present in `src/Notes.md`) file to keep track of data structure and details from the RFC. I have tried to mimic `dig` for output.

One issue i faced when implementing resolver in zig was lack of UDP support in `std.net`. I initially resorted to using [MasterQ32/zig-network](https://github.com/MasterQ32/zig-network) for using UDP to get the program going.
I did see in the spec that DNS should work over TCP, but missed the minute detail that we have to prefix the payload size in the payload, and I was just sending the actual DNS query over tcp, which lead to empty response from resolver.

After going again through any tcp related content in the spec, i understood the difference, and updated my code to use tcp from `std.net` and remove dependency on `zig-network`.

The current resolver supports only querying A record, and pretty-prints A, NS and SOA in response. The code is no way idiomatic. I plan to improve the code quality (adding tests, using comptime to remove redundant code, etc) and adding other resource record types from DNS spec over time.

## Things I liked about Zig

### comptime

Even though I have used `comptime` very little, it did feel very different from other compile time constructs from other language (like templates in cpp). Generic functions are very similar to writing normal functions.

I am yet to explore runtime-polyphormism and generic types, which should involve more comptime stuff to try.

### Testing included

One thing I liked very much about zig is "batteris included" for testing related functionality. Writing code for parsing bytes based on spec becomes a lot easier when you can add checks for the cases you are writing for.

This also allows TDD, which I was able to follow for some part of code when following the definition from the dns spec, writing test first on what is the expected behaviour and then able to write the actual implementation, leaving little/no room for mistakes.

### stdlib readabilty

Zig language is very readable/understandable, when you have understanding of general programming. I was able to understand some parts of stdlib just by going over their code (thanks zls for easy code navigation)

<!-- ### Tagged unions, enums and type system in general  -->

<!-- The type system is easy to understand, and easier to write code in. The  -->

## Some issues/gotchas

### Test source file coupling

Though testing functionality and batteries included for that is nice, having test code in the source files makes code readability a bit hard. When going through some code in `std.net`, this caused some issues while trying to understand how ip address gets converted to generic Address and then used for tcp connections.

While moving test to different files can cause issues (like visibility for non `pub` fns and how to test them), having them in between source code seems slightly bad.

### ZLS issues in vs code

I primarily use vs code for development, and zig language extension felt somewhat janky to use. Code completion for struct variables sometimes doesn't work, and it opening output terminal for showing error felt slightly uncomfortable to work with.

Though thinking that this is language support for still under development language, it is understandable to not have feature-rich perfect language server support. Hopefully ZLS gets better over time.

### Missing functionalities in stdlib

Some functionalities (like UDP network for my case) are missing from the stdlib. Though this is understandable for a developing language.

_Update_: While high level APIs are not available for UDP, we can use socket from [`std.os.socket`](https://github.com/ziglang/zig/blob/master/lib/std/os.zig#L3334) for using UDP (Thanks [@tauoverpi](https://github.com/tauoverpi) for pointing this out!). So we do have thin wrappers over "standard" functionalities in stdlib, just high level APIs are not present currently

### (gotcha) Struct custom formatting
Since I didn't go through chapter 2 from ziglearn (mistake), I didn't know about struct formatting. I saw `std.net.<type>` implemented `format` function, and tried replicating that for my structs for easy printing, but it following implmentation lead to program crashing (with exit code 1) without any reason on why it is crashing:

```zig
pub const header = struct {
    id: u16 = 0,
    flags: u16 = 0,

    pub fn format(
        self: header,
        comptime fmt: []const u8,
        options: std.fmt.FormatOptions,
        out_stream: anytype,
    ) !void {
        if (fmt.len != 0) std.fmt.invalidFmtError(fmt, self);
        _ = options;
        try std.fmt.format(out_stream, "{any}\n", .{self});
    }
}
```

Previously i was printing using `std.debug.print("{any}\n", header_var);` which worked fine, so understanding why this change was causing issues took me some time to debug.

Since we have implemented custom formatting, the formatter will be using our formatting when printing header variable, but as we are trying to format that within our custom formatting, it is leading to stack overflow?

I am unsure why there wasn't any error message (and build worked fine). Maybe diagnostics need improvement on crash.

After updating the formatting to print the variables instead of the object itself (`try std.fmt.format(out_stream, "{d} {d}\n", .{self.id, self.flags});`), the crash stopped.

## What next?

Though `resolv` currently works for simple A record query, there are lots of things which I can improve. Allowing custom nameserver, other queries, ipv6, dnssec are few of them. I will try to increase the functionality/options available in `resolv` over time. Hopefully, I will learn more about zig during that journey.

## Resources/Links

I found finding issues of zig slightly difficult in stack overflow, and rust related questions frequently popping up when searching for zig usage (even including "zig" in the search term).

Following are the resuorces I found helpful while learning and trying to work with zig:

- [https://www.huy.rocks/everyday/01-04-2022-zig-strings-in-5-minutes](https://www.huy.rocks/everyday/01-04-2022-zig-strings-in-5-minutes): String manipulation and working with strings
- [https://ziglang.org/documentation/master/](https://ziglang.org/documentation/master/): zig documentation, useful when finding specific keywords/types
- [https://ziglearn.org/](https://ziglearn.org/): Getting started with zig
- [https://github.com/ratfactor/ziglings](https://github.com/ratfactor/ziglings): Project having tiny broken programs, help in learning concepts learnt from ziglearn
