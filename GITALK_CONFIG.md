# Gitalk 评论系统配置指南

## 第一步：创建 GitHub OAuth App

1. 登录 GitHub，访问：https://github.com/settings/applications/new

2. 填写应用信息：
   - **Application name**: LiCode Blog Comments
   - **Homepage URL**: https://lichcode0114.github.io
   - **Application description**: LiCode技术博客评论系统
   - **Authorization callback URL**: https://lichcode0114.github.io

3. 点击 "Register application"

4. 记录下：
   - **Client ID**
   - **Client Secret**（点击 Generate a new client secret 生成）

## 第二步：修改配置

编辑 `themes/moment-grid/layout/post.ejs` 文件，找到以下代码并替换：

```javascript
const gitalk = new Gitalk({
  clientID: 'YOUR_GITHUB_CLIENT_ID',      // 替换为你的 Client ID
  clientSecret: 'YOUR_GITHUB_CLIENT_SECRET', // 替换为你的 Client Secret
  repo: 'lichcode0114.github.io',         // 你的 GitHub 仓库名
  owner: 'lichcode0114',                   // 你的 GitHub 用户名
  admin: ['lichcode0114'],                 // 管理员（你的 GitHub 用户名）
  id: location.pathname,
  distractionFreeMode: false,
  language: 'zh-CN'
})
```

## 第三步：准备收款二维码

1. 准备一张微信/支付宝收款二维码图片
2. 保存为 `reward-qr.jpg`
3. 放到 `source/images/` 目录

如果暂时没有，页面会显示"暂未设置收款码"

## 功能说明

### 1. 支持博主
- 顶部卡片展示收款二维码
- 心形图标带有跳动动画
- 吸引读者支持

### 2. 文章互动
- **点赞**：点击增加计数，再次点击取消
- **收藏**：收藏文章到本地
- **分享**：支持原生分享或复制链接
- 状态保存在本地浏览器

### 3. GitHub 评论
- 使用 GitHub Issues 作为评论后端
- 用户需要 GitHub 账号登录
- 支持 Markdown 语法
- 管理员可以删除评论

## 注意事项

1. 评论系统需要用户有 GitHub 账号
2. Client Secret 不要泄露到公开仓库
3. 每篇文章会在你的仓库创建一个对应的 Issue
4. 第一次访问文章时，管理员需要初始化评论

## 测试

生成并访问文章页面：
```bash
hexo clean && hexo g && hexo s
```

访问任意文章查看右侧功能是否正常显示。
