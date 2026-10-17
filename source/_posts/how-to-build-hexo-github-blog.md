---
title: 如何建立 Hexo + GitHub 的个人博客
date: 2026-07-07 10:00:00
tags: 
  - Hexo
  - GitHub
  - 博客搭建
  - 教程
categories: 
  - 技术教程
description: 从零开始搭建基于 Hexo 和 GitHub Pages 的个人技术博客，包含完整的安装、配置、部署流程。
---

## 前言

Hexo 是一个快速、简洁且高效的博客框架，使用 Markdown 解析文章，只需几秒就能生成静态网页。结合 GitHub Pages，我们可以免费搭建一个属于自己的技术博客。

本文将详细介绍如何从零开始搭建一个 Hexo + GitHub Pages 的个人博客。

## 准备工作

### 环境要求

在开始之前，需要确保你的电脑上安装了以下工具：

- **Node.js**（建议 14.0 以上版本）
- **Git**
- 一个 **GitHub 账号**

### 安装 Node.js

访问 [Node.js 官网](https://nodejs.org/) 下载并安装适合你操作系统的版本。安装完成后，在命令行中验证：

```bash
node -v
npm -v
```

### 安装 Git

访问 [Git 官网](https://git-scm.com/) 下载并安装。验证安装：

```bash
git --version
```

## 第一步：安装 Hexo

使用 npm 全局安装 Hexo 命令行工具：

```bash
npm install -g hexo-cli
```

安装完成后验证：

```bash
hexo -v
```

## 第二步：初始化博客

创建一个新的文件夹作为博客目录，然后初始化 Hexo：

```bash
hexo init blog
cd blog
npm install
```

初始化完成后，博客目录结构如下：

```
blog/
├── _config.yml          # 网站配置文件
├── package.json         # 应用程序信息
├── scaffolds/           # 模板文件夹
├── source/              # 资源文件夹
│   └── _posts/          # 文章存放目录
└── themes/              # 主题文件夹
```

## 第三步：本地预览

启动本地服务器：

```bash
hexo server
```

或者使用简写命令：

```bash
hexo s
```

打开浏览器访问 `http://localhost:4000`，你应该能看到默认的 Hexo 博客页面。

## 第四步：创建 GitHub 仓库

1. 登录 GitHub，创建一个新的仓库
2. 仓库名称必须是：`你的用户名.github.io`
3. 设置为 Public（公开）
4. 不要初始化 README

例如，如果你的 GitHub 用户名是 `zhangsan`，那么仓库名就是 `zhangsan.github.io`。

## 第五步：配置 SSH 密钥

为了方便部署，建议配置 SSH 密钥。

### 生成 SSH 密钥

```bash
ssh-keygen -t rsa -C "your_email@example.com"
```

一路回车使用默认设置。

### 添加到 GitHub

1. 复制公钥内容：

```bash
cat ~/.ssh/id_rsa.pub
```

2. 登录 GitHub，进入 Settings → SSH and GPG keys → New SSH key
3. 粘贴公钥内容并保存

### 测试连接

```bash
ssh -T git@github.com
```

看到 "Hi username! You've successfully authenticated..." 表示配置成功。

## 第六步：配置 Hexo 部署

编辑博客根目录的 `_config.yml` 文件，找到 `deploy` 部分：

```yaml
deploy:
  type: git
  repo: git@github.com:你的用户名/你的用户名.github.io.git
  branch: gh-pages
```

安装 Hexo 的 Git 部署插件：

```bash
npm install hexo-deployer-git --save
```

## 第七步：编写第一篇文章

创建新文章：

```bash
hexo new "我的第一篇博客"
```

这会在 `source/_posts/` 目录下生成一个 Markdown 文件。使用你喜欢的编辑器打开并编辑：

```markdown
---
title: 我的第一篇博客
date: 2026-07-07 10:00:00
tags: 
  - 随笔
categories: 
  - 生活
---

这是我的第一篇博客文章！

## 标题

正文内容...
```

## 第八步：部署到 GitHub Pages

### 生成静态文件

```bash
hexo generate
```

或使用简写：

```bash
hexo g
```

### 部署

```bash
hexo deploy
```

或使用简写：

```bash
hexo d
```

也可以一次性完成清理、生成和部署：

```bash
hexo clean && hexo g && hexo d
```

部署完成后，访问 `https://你的用户名.github.io` 即可看到你的博客。

## 第九步：个性化配置

### 修改网站信息

编辑 `_config.yml`：

```yaml
# Site
title: 我的技术博客
subtitle: '记录学习与成长'
description: '专注于前端开发、算法学习'
keywords: 编程,前端,算法
author: Your Name
language: zh-CN
timezone: 'Asia/Shanghai'
```

### 安装主题

Hexo 有丰富的主题生态。例如安装 NexT 主题：

```bash
cd blog
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

然后在 `_config.yml` 中修改：

```yaml
theme: next
```

## 常用命令总结

```bash
hexo new "文章标题"       # 新建文章
hexo new page "页面名"    # 新建页面
hexo generate            # 生成静态文件
hexo server              # 启动本地服务器
hexo deploy              # 部署到远程
hexo clean               # 清除缓存文件
hexo g -d                # 生成并部署
```

## 写作技巧

### Front-matter

每篇文章开头的 YAML 格式配置：

```yaml
---
title: 文章标题
date: 2026-07-07 10:00:00
updated: 2026-07-07 12:00:00
tags: 
  - 标签1
  - 标签2
categories: 
  - 分类
description: 文章摘要
cover: /images/cover.jpg
---
```

### Markdown 语法

Hexo 支持标准的 Markdown 语法，包括：

- 标题：`# H1` ~ `###### H6`
- 列表：`-` 或 `1.`
- 链接：`[文字](URL)`
- 图片：`![alt](URL)`
- 代码块：用三个反引号包裹
- 引用：`>`

## 进阶配置

### 配置域名

如果你有自己的域名，可以在 `source/` 目录下创建 `CNAME` 文件，内容为你的域名：

```
blog.example.com
```

然后在域名服务商处添加 CNAME 记录指向 `你的用户名.github.io`。

### 添加评论系统

可以集成 Gitalk、Valine、Disqus 等评论系统。

### 添加统计

- Google Analytics
- 百度统计
- 不蒜子访问统计

### SEO 优化

安装 SEO 插件：

```bash
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save
```

在 `_config.yml` 中配置：

```yaml
sitemap:
  path: sitemap.xml
baidusitemap:
  path: baidusitemap.xml
```

## 常见问题

### 部署后页面样式丢失

检查 `_config.yml` 中的 `url` 和 `root` 配置是否正确：

```yaml
url: https://你的用户名.github.io
root: /
```

### 文章不显示

- 检查文章的 `date` 是否是未来时间
- 运行 `hexo clean` 清除缓存后重新生成

### GitHub Pages 未更新

- 等待 1-2 分钟，GitHub Pages 需要构建时间
- 检查仓库的 Settings → Pages 中是否正确配置了分支

## 总结

通过以上步骤，你已经成功搭建了一个基于 Hexo + GitHub Pages 的个人博客。接下来你可以：

1. 定期发布高质量的技术文章
2. 自定义主题和样式
3. 添加更多功能插件
4. 优化 SEO 提升访问量

记住，坚持写作才是博客的核心。祝你的博客之旅愉快！

## 参考资源

- [Hexo 官方文档](https://hexo.io/zh-cn/docs/)
- [GitHub Pages 文档](https://docs.github.com/cn/pages)
- [Markdown 语法指南](https://markdown.com.cn/)
- [Hexo 主题列表](https://hexo.io/themes/)
