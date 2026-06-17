# RSS 订阅服务

基于 Cloudflare Workers 的 RSS 订阅与聚合服务，使用 GitHub OAuth 做访问控制，订阅源元数据存储在 Cloudflare KV，聚合后的内容存储在 Cloudflare R2。

## 功能

- GitHub OAuth 登录与白名单授权
- RSS 订阅源管理
- 定时抓取并聚合多路 RSS 内容
- 公开 RSS 内容接口
- 受保护的管理接口

## 技术栈

- Cloudflare Workers
- Hono
- TypeScript
- rss-parser
- Cloudflare KV
- Cloudflare R2
- Wrangler

## 项目结构

- `src/index.ts`：Worker 主入口、路由、认证、中间件、定时抓取逻辑
- `src/types.ts`：共享类型与 Cloudflare 绑定类型
- `public/`：静态资源
- `wrangler.jsonc`：Worker 配置、绑定、Cron

## 环境变量与绑定

Worker 代码当前依赖以下绑定和环境变量：

- `RSS_FEEDS`
- `RSS_BUCKET`
- `GITHUB_CLIENT_ID`
- `GITHUB_CLIENT_SECRET`
- `ALLOWED_GITHUB_USERS`
- `APP_URL`

## 本地开发

1. 安装依赖

```bash
npm install
```

2. 启动本地开发

```bash
npm run dev
```

3. 类型检查

```bash
npx tsc --noEmit
```

## 部署配置

1. 在 GitHub 创建一个 OAuth App
2. 将回调地址设置为 `https://<your-worker-domain>/auth/github/callback`
3. 在 Cloudflare Workers 中配置环境变量
4. 在 `wrangler.jsonc` 中配置 KV、R2 和 Cron 触发器

示例环境变量：

```env
GITHUB_CLIENT_ID=your_github_client_id
GITHUB_CLIENT_SECRET=your_github_client_secret
ALLOWED_GITHUB_USERS=user1,user2
APP_URL=https://your-worker-domain
```

## API 文档

### 公开 API

- `GET /api/rss/public`：获取全部 RSS 内容
- `GET /api/rss/public?author=xxx`：按作者筛选 RSS 内容

### 认证 API

- `GET /api/feeds`：获取订阅源列表
- `POST /api/feeds`：添加新订阅源
- `DELETE /api/feeds/:url`：删除订阅源
- `GET /api/rss`：获取 RSS 内容，需要 GitHub 登录
- `GET /api/rss?author=xxx`：按作者筛选 RSS 内容，需要 GitHub 登录
- `POST /api/rss/refresh`：手动刷新 RSS 内容
- `POST /api/cron/test`：手动触发一次定时抓取测试

## 说明

- 订阅源列表存储在 KV 的 `feeds` 键下
- 聚合内容存储在 R2 的 `rss.json` 对象中
- 公共内容接口不会要求登录
- 管理接口会校验 `ALLOWED_GITHUB_USERS` 白名单
