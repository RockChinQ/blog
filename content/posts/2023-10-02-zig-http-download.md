---
title: "Zig Tricks | Zig 语言实现 http 下载文件"
date: 2023-10-02
draft: false
---

\\n

最近看了看 Zig
语言，前几个月就有稍微了解一下，现在双节放假就实际上学了一下，看看能不能把
QChatGPT 的安装器用这个语言重构。当然，更多的还是对探索新技术的热爱。

\\n\\n\\n\\n

Zig 在上半年刚刚 builtin 了 http
客户端，看起来方便了很多，目前在网上搜索 "Zig http请求" "zig
下载http文件"都找不到任何示例，那么我就来贡献一篇。

\\n\\n\\n\\n

参考：

\\n\\n\\n\\n

[https://zigbyexample.github.io/read_write_file](https://zigbyexample.github.io/read_write_file){data-type="link"
data-id="https://zigbyexample.github.io/read_write_file"}

\\n\\n\\n\\n

[https://zigbyexample.github.io/http-client](https://zigbyexample.github.io/http-client){data-type="link"
data-id="https://zigbyexample.github.io/http-client"}

\\n\\n\\n\\n

当然还有[官方文档](https://ziglang.org/documentation/master/std/#A;std:http.Client){data-type="link"
data-id="https://ziglang.org/documentation/master/std/#A;std:http.Client"}

\\n\\n\\n\\n

本来还有问了GPT的，但是GPT生成的答案完全就是错的（21年还没有 builtin
http client）。

\\n\\n\\n\\n

Zig版本：0.11.0

\\n\\n\\n\\n

### 开始 {#开始 .wp-block-heading}

\\n\\n\\n\\n

你至少得读完了 <https://ziglearn.org/chapter-1/> 这一篇文章才适合看。

\\n\\n\\n\\n

这个写得水平其实也很差劲的，毕竟才学了三四天。

\\n\\n\\n\\n

``` wp-block-code
// 引入所需组件\nconst std = @import("std");\nconst allocator = std.testing.allocator;\nconst http = @import("std").http;\n\n// 实际操作函数\npub fn downloadFile(url: []const u8, target: []const u8) !void {\n    // 创建http客户端实例\n    var client = http.Client{ .allocator = allocator };\n    defer client.deinit();  // 函数退出时释放http客户端资源\n\n    // 从客户端创建请求体\n    // Method: 去看http.Method支持的enums\n    // Uri: 从std.Uri.parse把地址字符串转换成Uri实例\n    // Allocator:\n 直接使用std.testing.allocator，这里我也不太清楚，事实上用 std.heap.page_allocator 之类的应该也可以\n    // Options: 给空结构体\n    var request = try client.request(http.Method.GET, try std.Uri.parse(url), .{ .allocator = allocator }, .{});\n    defer request.deinit();  // 函数退出时释放req的资源\n    try request.start();  // 执行请求\n    \n    // 这个函数的docstring说的是等待请求完全响应完成\n    // 但是文件下载应该要流式进行？但是不wait直接取response会error\n    // ps：这里可能是等待请求发送完成\n    try request.wait();\n\n    // 取到response\n    var response = request.response;\n\n    // 校验response的status\n    if (response.status != http.Status.ok) {\n        return error.FailedToDownloadFile;\n    }\n\n    // 创建文件\n    var outputFile = try std.fs.cwd().createFile(target, .{\n        .truncate = true, // 这里强制覆盖已存在的此文件的所有内容，具体参数请查看文档 std.fs.File.CreateFlags 的参数\n    });\n    defer outputFile.close();  // 不赘述\n\n    // 分段接收响应体\n    while (true) {\n        var buffer: [4096]u8 = undefined;\n        var bytesRead = try request.read(buffer[0..]);  // 这里还是用request对象去取响应体\n        if (bytesRead == 0) break;\n\n        // 写文件\n        _ = try outputFile.writer().write(buffer[0..bytesRead]);\n    }\n}\n\ntest "download file" {\n    try downloadFile("https://user-images.githubusercontent.com/37870767/239670001-06e64501-02fb-4d4a-ab6f-cf18d8638ace.png", "test.png");\n}\n
```

\\n
