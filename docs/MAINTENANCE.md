# 站点维护与更新

本文记录当前 `blog.agisj.cn` 这套部署的日常更新方法。

## 当前部署结构

- 本地目录：`D:\Codex_Project\qiaomu-blog`
- 你的 GitHub 仓库：`https://github.com/mmmbmmm/qiaomu-blog.git`
- 原作者上游仓库：`https://github.com/joeseesun/qiaomu-blog-opensource.git`
- 生产域名：`https://blog.agisj.cn`
- 部署方式：推送到 `origin/main` 后，GitHub Actions 自动部署到 Cloudflare Workers

远端含义：

```text
origin   = 你的仓库，用来保存自己的版本并触发自动部署
upstream = 原作者仓库，用来拉取后续开源更新
```

## 更新自己的博客代码

```powershell
cd D:\Codex_Project\qiaomu-blog
git pull origin main

# 修改代码后
git add .
git commit -m "你的更新说明"
git push origin main
```

推送到 `main` 后，GitHub Actions 会自动部署到 Cloudflare，不需要手动运行 `npm run deploy`。

## 同步原作者上游更新

```powershell
cd D:\Codex_Project\qiaomu-blog
git fetch upstream
git checkout main
git pull origin main
git merge upstream/main
npm run build
git push origin main
```

如果 `git merge upstream/main` 出现冲突：

```powershell
# 先手动解决冲突文件
git add .
git commit
npm run build
git push origin main
```

## 更新管理员密码或 AI 配置

敏感信息只放在本机 `.deploy.env`，不要提交到 Git。

修改 `.deploy.env` 后，需要同步到两处：

1. GitHub Actions Secrets：保证以后重新部署仍使用新值。
2. Cloudflare Worker Secrets：保证线上立即生效。

常用键名：

```env
ADMIN_PASSWORD=后台登录密码
AI_API_KEY=外部 AI 服务 Key
AI_BASE_URL=外部 AI 服务地址
AI_MODEL=默认 AI 模型
CLOUDFLARE_ACCOUNT_ID=Cloudflare 账号 ID
CLOUDFLARE_API_TOKEN=Cloudflare 部署 Token
GITHUB_TOKEN=GitHub Token
```

如果不想手动同步，可以直接让 Codex：

```text
把 .deploy.env 里的管理员密码同步到线上
```

或：

```text
把 .deploy.env 里的 AI 配置同步到线上并重新部署
```

## 数据与代码更新的关系

文章、图片、后台设置主要存放在 Cloudflare D1 / R2 中，不会因为推送代码或重新部署 Worker 而丢失。

需要额外注意的是：如果上游更新包含数据库 schema 变化，应先检查 `db/schema.sql` 或迁移说明，再决定是否对远程 D1 执行 migration。

## 快速检查

查看本地仓库状态：

```powershell
git status --short --branch
```

查看最近提交：

```powershell
git log --oneline -5
```

检查线上页面：

```powershell
Invoke-WebRequest -Uri "https://blog.agisj.cn" -Method Head
Invoke-WebRequest -Uri "https://blog.agisj.cn/admin" -Method Head
```

