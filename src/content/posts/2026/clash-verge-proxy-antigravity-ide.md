---
title: Clash Verge 进阶教程：使用正则筛选节点优化 Google Antigravity IDE 网络体验
published: 2026-01-14
description: '介绍如何使用 Clash Verge Rev 的 Script（脚本）功能，通过正则表达式从你的机场订阅中筛选出特定地区（如美国、日本、台湾）的优质节点，创建一个专用的策略组，并强制 Antigravity IDE 走这个专用通道。'
tags: ['Proxy', 'FuckGFW', 'IDE']
category: '技术分享'
draft: false
lang: 'zh-CN'
---

无论是在进行云端开发还是使用先进的 AI 辅助编程工具（如 Google Antigravity IDE），一个稳定且低延迟的网络环境都是至关重要的。很多时候，我们不仅需要“能连上”，还需要“连得快、连得稳”。

本文将介绍如何利用 Clash Verge Rev 强大的 Script（脚本）功能，通过正则表达式从你的机场订阅中筛选出特定地区（如美国、日本、台湾）的优质节点，创建一个专用的策略组，并强制 Antigravity IDE 走这个专用通道。

## 为什么需要这样做？

1.  **节点筛选**：机场订阅通常包含大量节点，质量参差不齐。对于 AI 服务，通常美国、日本或台湾的节点支持更好。机场配置文件下发后需要二次处理，筛选出优质节点。
2.  **隔离流量**：创建一个独立的 "Gemini" 策略组，可以避免其他无关流量挤占带宽，也可以让你单独为 IDE 切换节点，而不影响浏览器看视频等其他操作。
3.  **进程级路由**：通过进程名（Process Name）精准分流，无需手动配置 IDE 的代理设置，实现“无感”加速。

## 配置步骤

### 1. 准备工作

确保你已经安装了 [Clash Verge Rev](https://github.com/clash-verge-rev/clash-verge-rev) 并配置好了基础的机场订阅。

### 2. 编辑脚本

1.  打开 **Clash Verge Rev**。
2.  进入左侧的 **Profiles (配置)** 菜单。
3.  找到你当前使用的配置文件（或者新建一个 New Script）。
4.  右键点击配置文件，选择 **Edit Script (编辑脚本)**；或者点击界面右上角的 "Script" 按钮（确保 Enable Script 选项已开启）。

### 3. 写入脚本代码

在编辑器中输入以下代码。这段代码主要做了三件事：创建带正则筛选的新策略组、定义进程分流规则、将新规则应用到配置中。

```javascript
// Define main function (script entry)
function main(config) {
  // ============================================
  // 1. 定义新的策略组 "Gemini" (带筛选功能)
  // ============================================
  const geminiGroup = {
    name: "Gemini",
    type: "select", // 这里使用 select 手动选择，也可以改为 "url-test" 让其自动选择延迟最低的节点

    // ⛔️ 注意：这里的 "zhs", "fczs" 是示例中的 Proxy Provider 名称
    // 你需要根据你自己的订阅名称进行修改，或者删除 "use" 字段，改用 filter 筛选全局节点
    // 如果你的节点都在全局池里，可以不写 use，Clash 会默认在所有节点中筛选
    // 因为我合并了两个机场的配置文件，所以这里需要指定
    use: ["zhs", "fczs"],

    // 【核心修改】在这里添加正则筛选
    // 只有名字里包含 日本、台湾、美国 (及其常见英文简写) 的节点才会出现在这个组里
    // (?i) 表示忽略大小写，| 表示或
    filter: "(?i)(日本|Japan|JP|台湾|台灣|TW|美国|US|United States)",

    // 你仍然可以保留 DIRECT 作为备选，万一节点都挂了可以选直连（虽然通常连不上）
    proxies: ["DIRECT"],
  };

  // 确保 proxy-groups 存在，防止配置为空报错
  if (!config["proxy-groups"]) {
    config["proxy-groups"] = [];
  }

  // 将 Gemini 组插入到策略组列表的第一个，方便在面板最上方操作
  config["proxy-groups"].unshift(geminiGroup);

  // ============================================
  // 2. 定义分流规则
  // ============================================

  // (可选) 微信等国内应用走直连，防止卡顿
  const wechatRules = [
    "PROCESS-NAME,WeChat,DIRECT"
  ];

  // Antigravity IDE 及相关 AI 进程强制走 Gemini 策略组
  const geminiRules = [
    "PROCESS-NAME,Antigravity,Gemini", // IDE 主进程
    "PROCESS-NAME,Antigravity Helper,Gemini", // 辅助进程
    "PROCESS-NAME,language_server_macos_arm,Gemini", // 语言服务（Mac ARM）
    "PROCESS-NAME,Google Gemini,Gemini", // Gemini 相关服务
  ];

  // ============================================
  // 3. 合并规则
  // ============================================

  // 读取原有的规则 (防止报错，如果为空则初始化为空数组)
  const oldRules = config.rules || [];

  // 将新规则合并到旧规则之前 (Prepend)，让我们的规则优先级最高
  // 顺序：微信规则 -> Gemini 规则 -> 原有规则
  config.rules = [...wechatRules, ...geminiRules, ...oldRules];

  // 4. 返回修改后的配置
  return config;
}
```

### 4. 代码核心原理解析

#### 正则表达式筛选 (`filter`)

```javascript
filter: "(?i)(日本|Japan|JP|台湾|台灣|TW|美国|US|United States)";
```

这行代码是整个配置的灵魂。

- `(?i)`: 开启大小写不敏感模式，匹配 `Japan` 或 `japan` 都可以。
- `(A|B|C)`: 匹配 A 或 B 或 C。
- 通过这个设置，Clash 会自动遍历你订阅中的所有节点，把名字里带有“日本”、“美国”等关键词的节点“抓”进这个 `Gemini` 策略组里。

#### 进程名匹配 (`PROCESS-NAME`)

```javascript
"PROCESS-NAME,Antigravity,Gemini";
```

这是 Clash 的高级规则类型。它不看域名，不看 IP，只看发起网络请求的程序名字。只要是 Antigravity 发起的请求，一律扔给 `Gemini` 策略组处理。这比配置复杂的域名列表要省心得多且不易出错。

## 验证是否生效

1.  保存脚本并开启配置。
2.  点击 Clash Verge 侧边栏的 **Proxies (代理)**。
3.  你应该能看到一个新的策略组叫 **Gemini**。
4.  展开 **Gemini**，你会发现里面只剩下了日本、美国、台湾的节点，其他地区的节点都被过滤掉了。
5.  打开 Antigravity IDE，在 **Connections (连接)** 面板中观察，这时候相关的流量应该会显示 `Rule: Gemini`。

## 总结

通过这段简单的脚本，我们不仅为 Antigravity IDE 打造了一条“VIP 专线”，还学会了如何利用 Clash Verge 的 Script 功能进行复杂的节点筛选和路由控制。这套逻辑同样适用于其他需要特定网络环境的应用，只需修改 `PROCESS-NAME` 即可。
