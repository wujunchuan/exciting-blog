# 在Astro博客中使用Giscus评论系统

本指南将帮助您在Astro博客中集成Giscus评论系统。

## 1. 配置Giscus

### 1.1 设置GitHub Discussions

首先，您需要在您的GitHub仓库中启用Discussions功能：

1. 进入您的GitHub仓库
2. 点击"Issues"标签页
3. 点击"Discussions"选项卡
4. 点击"Set up discussions"按钮启用该功能

### 1.2 配置Giscus应用

访问 [Giscus官网](https://giscus.app/) 进行配置：

1. 在页面上点击"Enable Giscus"按钮
2. 授权"Giscus"应用访问您的GitHub仓库
3. 选择您的仓库
4. 选择Discussion分类（通常选择"Announcements"或创建新的分类）
5. 设置默认评论位置行为（建议选择"Specific Discussion Number"或"URL pathname"）
6. 选择主题样式（与您的网站主题匹配）
7. 获取生成的代码片段，并记下所需的参数值

## 2. 配置文件设置

在 `src/config.ts` 文件中，更新 `giscusConfig` 对象的值：

```typescript
export const giscusConfig: GiscusConfig = {
    repo: 'your-username/your-repo', // 替换为您的GitHub仓库名
    repoId: 'your-repo-id',          // 从Giscus app获取的仓库ID
    category: 'Announcements',       // GitHub Discussions分类名
    categoryId: 'your-category-id',  // 从Giscus app获取的分类ID
    mapping: 'pathname',
    theme: 'light',
    reactionsEnabled: '1',
    emitMetadata: '0',
    inputPosition: 'top',
    lang: 'en',
    loading: 'lazy'
};
```

## 3. 验证集成

1. 重启开发服务器以应用更改
2. 访问博客文章页面，确认评论系统已正确显示
3. 测试评论功能，确保其正常工作

## 4. 高级配置

### 4.1 主题集成

Giscus评论系统会自动跟随网站的主题（浅色/深色模式）。当用户切换主题时，评论系统也会相应更改。

### 4.2 评论显示控制

您可以在博客文章的Frontmatter中控制是否显示评论：

```markdown
---
title: "Hello World"
description: "This is my first post"
enableComments: false  // 设置为false可隐藏评论系统
---
```

## 5. 故障排除

- 如果Giscus评论系统无法加载，请检查您的配置参数是否正确
- 确保您的GitHub仓库已启用Discussions功能
- 确保您使用的分类ID和仓库ID是正确的
- 检查浏览器控制台是否有错误信息

## 6. 安全注意事项

- Giscus评论系统是基于GitHub Discussions的开源评论系统，无需额外服务器
- 评论需要GitHub账户才能发表，这有助于减少垃圾评论
- 所有评论内容都存储在您的GitHub仓库中