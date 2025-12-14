---
title: 使用 CloudFlare 部署共享剪切板
published: 2025-12-13
description: '介绍如何使用 CloudFlare Workers 部署一个功能丰富的共享剪切板服务，支持文本/图片上传、语法高亮、CLI 工具、Raycast 快捷操作等功能'
image: 'https://storage.fuckwechat.com/G6CVGeZbMAE8fan_yyyy12Su_220226.jpeg'
tags: ['cloudflare worker', 'serverless']
category: '技术分享'
draft: false
lang: 'zh-CN'
---

一直听说 CloudFlare 赛博活佛的大名，但之前没有使用需求。最近因业务需要尝试了这类 Serverless 服务，体验非常棒。

这里推荐一个简易的，与世界即时分享您的代码片段和文件的服务。

## 功能特色

我在原版基础上做了一些增强功能，主要包括：

1. **丰富的 CLI 工具**：提供了便捷的命令行工具，支持多种上传方式，包括直接文本输入、文件上传、从剪贴板读取内容（甚至支持图片！）。
2. **智能语法检测**：CLI 工具能自动识别代码语言类型，并应用相应的语法高亮。
3. **多平台剪贴板支持**：支持 macOS 和 Linux 平台的文本和图片剪贴板内容上传。
4. **完整 API 支持**：支持自定义过期时间、命名分享内容、设置访问密码等高级功能。
5. **安全性**：支持客户端加密、身份验证（Basic Auth）等安全措施。
6. **多格式支持**：支持代码语法高亮、Markdown 渲染、URL 缩短等功能。

## 部署过程

部署过程非常简单，按照官方文档即可完成。主要步骤如下：

1. 安装 `node` 和 `yarn`。
2. 在 Cloudflare 仪表板中创建 KV 命名空间和 R2 存储桶，记住其 ID。
3. 克隆仓库并进入目录。
4. 修改 `wrangler.toml` 中的相关配置项。
5. 登录 Cloudflare 并进行部署。

以下是关键的 `wrangler.toml` 配置文件样例：

:::important
虽然免费额度量大管饱，但还是建议个人使用要设置密码。
:::


```toml
name = "pb"
compatibility_date = "2025-04-24"

workers_dev = false
main = "worker/index.ts"

[[rules]]
type = "Text"
globs = [ "**/*.html", "**/*.md" ]
fallthrough = true

[assets]
directory = "dist/frontend"
run_worker_first = true
binding = "ASSETS"

[triggers]
# clean r2 garbage every day
crons = ["0 0 * * *"]

#----------------------------------------
# lines below are what you should modify
#----------------------------------------

[[routes]]
# Refer to https://developers.cloudflare.com/workers/wrangler/configuration/#routes
pattern = "paste.fuckwechat.com"
custom_domain = true

[[kv_namespaces]]
binding = "PB"  # do not touch this
id = "eca225171b584b55a388f7c447bfa948"  # id of your KV namespace

[[r2_buckets]]
binding = "R2"  # do not touch this
bucket_name = "pb-shz-al"  # bucket name of your R2 bucket

[vars]
# must be consistent with your routes
DEPLOY_URL = "https://paste.fuckwechat.com"

# url to repo, displayed in the index page
REPO = "https://github.com/wujunchuan/pastebin-worker"

# the page title displayed in index page
INDEX_PAGE_TITLE = "Pastebin Worker"

# the name displayed in TOS
TOS_MAINTAINER = "wujunchuan"

# the email displayed in TOS
TOS_MAIL = "admin@fuckwechat.com"

# Cache-Control max-age for static pages
CACHE_STATIC_PAGE_AGE = 7200

# Cache-Control max-age for paste pages
CACHE_PASTE_AGE = 600

# Default expiration
DEFAULT_EXPIRATION = "7d"

# Max expiration
MAX_EXPIRATION = "30d"

# Files larger than this threshold will be stored in R2
R2_THRESHOLD = "100K"

# File larger than this will be denied
R2_MAX_ALLOWED = "100M"

# The following mimetypes will be converted to text/plain
DISALLOWED_MIME_FOR_PASTE = ["text/html", "audio/x-mpegurl"]

[vars.BASIC_AUTH]
wujunchuan = "$2b$08$TA4lueuSHOb9wgrmMYdISuDwlj0Q.hK0XJyaMgxM6Hh6Y2rlAa5uy"
```

## 使用体验

经过部署后，你可以通过网站界面直接分享内容，也可以通过 API 进行程序化操作。特别值得一提的是，我还开发了一个高级的 bash 脚本 `advanced_paste.sh`，让命令行操作更加便捷：

```bash
# 上传一段文本
./advanced_paste.sh "Hello World"

# 设置过期时间和自定义名称
./advanced_paste.sh -e 24h -n ~myshare "Hello World"

# 上传文件并设置管理密码
./advanced_paste.sh -f /path/to/file.txt -s 123456

# 从剪贴板获取内容上传（支持文本和图片！）
./advanced_paste.sh -c

# 带用户认证的上传
./advanced_paste.sh -u wujunchuan -p your_setup_password "Hello World"
```

这个脚本可以自动检测操作系统，处理剪贴板内容，并且支持跨平台（macOS 和 Linux）。特别是对图片的支持，利用了各平台的剪贴板工具，使得截图分享变得异常简单。

整个方案充分利用了 Cloudflare Workers 的优势：
- **全球分发**：你的分享链接在全球范围内都能快速访问
- **按需付费**：免费额度对于个人使用完全够用
- **易于维护**：无需管理任何服务器
- **高可用**：基于 Cloudflare 的基础设施，稳定性极高

这种轻量级的分享服务非常适合开发者之间交换代码片段、配置文件，或者临时分享一些文本内容，避免了发送邮件附件的麻烦。

## 使用 Raycast 脚本快速接入

```sh
#!/bin/bash

# Required parameters:
# @raycast.schemaVersion 1
# @raycast.title paste clipboard to pastebin
# @raycast.author John Trump
# @raycast.mode compact

# Optional parameters:
# @raycast.icon 🤖

# 将剪贴板内容上传到pastebin
if [ "$(osascript -e 'the clipboard')x" != "x" ]; then
    # 如果剪贴板有内容，则从剪贴板上传
    # before use advanced_paste.sh, install it first
    # shell execute `install -Dm755 advanced_paste.sh ~/.local/bin`
    "$HOME/.local/bin/advanced_paste.sh" -u your_setup_username -p your_setup_password -c | jq -r '.url' | pbcopy
else
    # 否则提示用户输入内容
    echo '{"error": "剪贴板为空，请复制一些内容后再试"}'
fi
```

## 致敬原版

本项目基于 SharzyL 开发的 [pastebin-worker](https://github.com/SharzyL/pastebin-worker) 进行二次开发，在此表示感谢！原版项目设计精巧，充分利用了 Cloudflare Workers 的能力，为用户提供了一个简洁高效的分享服务。

::github{repo="SharzyL/pastebin-worker"}
::github{repo="wujunchuan/pastebin-worker"}
