---
title: Telegram 机器人初体验
published: 2021-07-17
description: '注册 Telegram 机器人并部署到 Vercel'
image: 'https://storage.fuckwechat.com/maxresdefault_yyyy12Su_174346.jpg'
tags: ['telegram', 'serverless']
category: 技术分享
draft: false
lang: 'zh-CN'
---

# Telegram 机器人

本文将演示如何注册 Telegram 机器人，并将其部署到 [Vercel](https://vercel.com/) 上。

之所以选用 Vercel 是因为它是一个 Serverless 的云服务提供商，我的 Notion 博客就是部署在上面的，四舍五入约等于白嫖。

## 步骤

安装 Vercel：

```bash
yarn install -g vercel
```

登录 Vercel：

```bash
vercel login
```

我们写个简单的 demo，来验证一下：

```javascript
// api/webhook.js
module.exports = (request, response) => {
  response.json({
    body: request.body,
    query: request.query,
    cookies: request.cookies,
  });
};
```

本地预览项目：`vercel dev`

访问 `http://localhost:3000/api/webhook?hello=world`，这里我们会看到返回了一段 JSON。到目前为止，这个和普通的 Node 项目并没有什么区别。

### 注册机器人

[@BotFather](https://t.me/botfather)

![注册机器人](https://i.loli.net/2021/07/17/1KaLDlj3ZvsYOdG.png)

### 正式编写代码

```javascript
// https://github.com/yagop/node-telegram-bot-api/issues/319#issuecomment-324963294
// Fixes an error with Promise cancellation
process.env.NTBA_FIX_319 = 'test';

const TelegramBot = require('node-telegram-bot-api');

module.exports = async (request, response) => {
  const bot = new TelegramBot(process.env.TELEGRAM_TOKEN);
  try {
    const { body } = request;
    if (body.message) {
      const {
        chat: { id },
        text,
      } = body.message;
      const message = `✅ 感谢您的留言：*"${text}"*\n祝您有美好的一天！ 👋🏻`;
      await bot.sendMessage(id, message, { parse_mode: 'Markdown' });
    }
  } catch (error) {
    // 如果发送消息时出现错误，则可以将其记录到 Vercel 控制台
    console.error('发送消息时出错');
    console.log(error.toString());
  }
  response.send('OK');
};
```

本地启动：`TELEGRAM_TOKEN=<YOUR-BOT-TOKEN> vercel dev`，Vercel 默认会在 3000 端口启动服务。

当您向机器人发送消息时，`telegram-bot-server` 会向其绑定的地址发起 POST 请求。这段代码就是在接收并处理 `telegram-bot-server` 的 POST 请求。

### 使用 ngrok 映射到外网

由于后续调用 Telegram API 注册 webhook 需要 `HTTPS` 的环境，并且需要一个外网可以访问的地址，我们可以使用 [ngrok](https://ngrok.com/) 将本地端口映射到外网，并且会分配一个 `HTTPS` 的公网地址。

安装方法很简单：`yarn install -g ngrok`

使用也很简单，比如我们项目的端口地址为 3000，我们想要将其暴露出去，只需要执行：

```bash
ngrok http 3000
```

> 抛砖引玉一下，这玩意很好用。假如你在工作中想要把本地跑的 Demo 给远方的朋友看，可以使用它来实现内网穿透。

![ngrok 界面](https://i.loli.net/2021/07/17/UiK2cTmj5xCVzRE.png)

### 注册 webhook 到机器人

注册 webhook 到机器人：

```bash
curl -X POST https://api.telegram.org/bot<YOUR-BOT-TOKEN>/setWebhook \
     -H 'Content-Type: application/json' \
     -d '{"url": "https://8fbd312cf3d7.ngrok.io/api/webhook"}'
```

![成功设置 webhook](https://i.loli.net/2021/07/17/hxlW3HQRrNPT8aI.png)

成功收到消息并回复信息。

## 部署到 Vercel

部署很简单，只需要执行 `vercel` 即可。你甚至可以关联到 GitHub 的仓库（这里需要将 vercel 机器人加入到仓库中），只要 master 分支有新的提交，就会自动部署。

配置环境变量到 Vercel：

`https://vercel.com/<your-vercel-name>/<your-project-name>/settings/environment-variables`

![配置环境变量](https://i.loli.net/2021/07/17/HRKf7pVXYSvswZa.png)

## 相关链接

- [@BotFather](https://t.me/botfather)
- [Telegram API 文档](https://core.telegram.org/bots/api)
