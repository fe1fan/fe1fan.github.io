+++
author = "Fe1Fan"
title = "Rust Gui 框架开发第一篇 -- 打开浏览器"
date = "2024-03-20"
description = ""
tags = [
    "rust",
]
categories = [
    "rust",
]
series = ["Eleven"]
+++

<!--more-->

![images](https://cdn.xka.io/c728fa49-2557-46b0-a12c-67dc38fea8af%2F54fd4d2e-3df3-410e-876e-83c610a9339b.png)

## 1. 开篇

在很早之前使用 golang 做桌面应用程序的时候发现了一个特别有意思的库 [lorca](https://github.com/zserge/lorca)，趁着学习
rust 的机会，准备复刻一个 rust 的版本出来，所以才有了今天的 `Eleven` 系列。

## 2. 基本原理

在现有的跨平台的各类语言的 GUI 开源库中，比较有名的是 [electron](https://github.com/electron/electron)，
它的工作原理很简单：使用一个浏览器内核来渲染页面，并使用一些 JavaScript 代码来控制页面的行为，然后在此基础上为 js 提供一些
系统级别的 API。

但是这个框架争议比较大，因为它每打包一个程序都会内嵌一个完整的浏览器内核，导致程序体积比较大，这样就会导致
在你的计算机上同时存在着不同版本的多个浏览器内核，让人感觉很不舒服。

而 Lorca 的工作原理相对简单一些，它默认使用你本地已安装的浏览器来渲染页面，它通过 WebSocket 实现前端的 JavaScript 与后端的 Golang 通信。

## 3. 准备工作

因为我们需要用户本地的浏览器作为载体，所以我们需要查看各个浏览器的启动参数，以达到启动后渲染页面并且执行我们入口代码的目的。

在 [这个网站](https://peter.sh/experiments/chromium-command-line-switches/)，里面有比较详细的参数，我们选中我们需要的一些来说明用法

这些是 Chrome 浏览器启动参数的解释：

1. `--user-data-dir=/tmp/chrome-temp`: 指定用户数据目录，Chrome 将在指定的目录中保存用户数据，例如书签、历史记录等。
2. `--window-size=$width,$height`: 设置启动时的窗口大小，其中 $width 和 $height 是窗口的宽度和高度。
3. `--window-position=$x,$y`: 设置窗口启动时的位置，其中 $x 和 $y 是窗口左上角的横坐标和纵坐标。
4. `--disable-gpu`: 禁用 GPU 加速，可能在一些环境下遇到 GPU 兼容性问题时使用。
5. `--remote-debugging-port=0`: 启用远程调试功能，端口设置为 0 表示使用随机端口。
6. `--disable-software-rasterizer`: 禁用软件渲染器，可能在一些环境下遇到渲染问题时使用。
7. `--disable-extensions`: 禁用扩展功能。
8. `--disable-sync`: 禁用同步功能，即禁止 Chrome 与其他设备同步数据。
9. `--disable-default-apps`: 禁用默认应用程序。
10. `--disable-translate`: 禁用翻译功能。
11. `--disable-logging`: 禁用日志记录。
12. `--disable-background-networking`: 禁用后台网络请求。
13. `--disable-notifications`: 禁用通知。
14. `--disable-popup-blocking`: 禁用弹出窗口阻止功能。
15. `--hide-scrollbars`: 隐藏滚动条。
16. `--mute-audio`: 静音音频。
17. `--no-first-run`: 不显示首次运行时的欢迎页面。
18. `--no-default-browser-check`: 不检查是否为默认浏览器。
19. `--disable-background-timer-throttling`: 禁用后台定时器限制。
20. `--disable-features=site-per-process`: 禁用站点隔离功能，这个功能将每个网站都分配给不同的进程。

这是选中的一部分的参数，后续根据需要会继续修改，这里面有一个比较重点的参数：

`--user-data-dir=/tmp/chrome-temp`

在 Chrome 中，当用户目录相同时被视为同一个 Session，为了不影响正常的浏览器行为，我们需要指定一个临时的启动目录。

## 4. 代码实现

这里我们经过上面的准备工作，我们可以开始准备我们的第一行代码，从打开浏览器开始。

```rust
use std::os::unix::prelude::CommandExt;
use std::process::Command;

const BROWSERS: [&str; 1] = [
    "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome",
    //Other Browser Path
];

const ARGS: [&str; 18] = [
    "--user-data-dir=/tmp/chrome-temp",
    "--disable-gpu",
    "--remote-debugging-port=0",
    "--disable-software-rasterizer",
    "--disable-extensions",
    "--disable-sync",
    "--disable-default-apps",
    "--disable-translate",
    "--disable-logging",
    "--disable-background-networking",
    "--disable-notifications",
    "--disable-popup-blocking",
    "--hide-scrollbars",
    "--mute-audio",
    "--no-first-run",
    "--no-default-browser-check",
    "--disable-background-timer-throttling",
    "--disable-features=site-per-process",
];

fn main() {
    let mut command = Command::new(BROWSERS[0]);
    command.args(&ARGS);
    command.arg("--app=".to_owned() + "https://example.com");
    command.exec();
}
```

执行上面代码后，就得到了文章开头的效果。

我们继续优化上面的代码，需要考虑到不同的操作系统的浏览器安装位置不同，由此我们把上面的代码
整理成一个统一的方法，如下:

```rust
use std::{fs, thread};
use std::process::Command;

#[cfg(target_os = "macos")]
const BROWSERS: [&str; 6] = [
    "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome",
    "/Applications/Chromium.app/Contents/MacOS/Chromium",
    "/usr/bin/google-chrome-stable",
    "/usr/bin/google-chrome",
    "/usr/bin/chromium",
    "/usr/bin/chromium-browser",
];

#[cfg(target_os = "windows")]
const BROWSERS: [&str; 1] = [
    "C:\\Program Files (x86)\\Google\\Chrome\\Application\\chrome.exe",
];

#[cfg(target_os = "linux")]
const BROWSERS: [&str; 1] = [
    "/usr/bin/google-chrome-stable",
];


// The command line arguments for Chrome.
const ARGS: [&str; 21] = [
    //TODO add support other platforms
    "--user-data-dir=/tmp/chrome-temp",
    "--window-size=$width,$height",
    "--window-position=$x,$y",
    "--disable-gpu",
    "--remote-debugging-port=0",
    "--disable-software-rasterizer",
    "--disable-extensions",
    "--disable-sync",
    "--disable-default-apps",
    "--disable-translate",
    "--disable-logging",
    "--disable-background-networking",
    "--disable-notifications",
    "--disable-popup-blocking",
    "--hide-scrollbars",
    "--mute-audio",
    "--no-first-run",
    "--no-default-browser-check",
    "--disable-background-timer-throttling",
    "--disable-features=site-per-process",
    "--app=$url",
];

pub(crate) fn open_browser(url: &str, width: u32, height: u32, x: u32, y: u32) {
    println!("Opening browser with URL: {}", url);
    // find the first browser that exists.
    let browser = BROWSERS.iter().find(|&&b| fs::metadata(b).is_ok());
    if let Some(browser) = browser {
        let args = ARGS.iter().map(|arg| {
            arg.replace("$width", &width.to_string())
                .replace("$height", &height.to_string())
                .replace("$x", &x.to_string())
                .replace("$y", &y.to_string())
                .replace("$url", url)
        });
        let mut command = Command::new(browser);
        command.args(args);
        match command.spawn() {
            Ok(_) => {
                println!("Browser launched successfully.");
            },
            Err(e) => println!("Failed to launch browser: {}", e),
        }
        thread::sleep(std::time::Duration::from_secs(10));
    } else {
        println!("No browser found");
    }
}
```

我们需要统一的参数暂时有：url, width, height, x, y，这样我们在 App 启动初期就可以通过调用这些方法来绘制页面。

## 5. 后续计划

这里我们成功的打开了浏览器，让他看起来像个真正的应用，后续我们的大概思考运行流程是这样的：

1. 读取用户配置 package.toml 确定启动的页面。
2. 预加载基础能力的 JS 文件，这个 JS 文件会提供一些 Native API 用来与 Rust 通信或者直接调用系统 API。
3. JS 建立 WebSocket 与 Rust 通信。
4. 载入用户的入口 Html 文件，加载静态资源等。
5. 完成。

从这里我们可以看到，我们后续需要做的一些事情，这里我们先把这些列出来，后续我们会逐步实现。