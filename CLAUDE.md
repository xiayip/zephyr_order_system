# 项目说明 (Claude Code Memory)

末端配送机器人 VR 遥操 + 订单系统。当前阶段:**M1 单机器人订单贯通**。

完整架构、决策、schema 见 Claude Project knowledge。本文档只放仓库内执行相关信息(目录、命令、约定)。

## 仓库结构

```
.
├── robot/          # 机器人端代码 (ROS2 + Python)
├── web/            # 订单中心 (Next.js + supabase-js)
├── token-server/   # LiveKit token 签发服务(沿用现有)
├── supabase/       # 数据库迁移 + Edge Functions
│   ├── migrations/
│   └── functions/
└── docs/           # 仓库内备份的项目文档(可选)
```

(随实际开发结构调整)

## 技术栈

- 数据库 / 实时 / 鉴权:Supabase (Postgres 15)
- Web 前端:Next.js 14 (App Router) + TypeScript + supabase-js
- 机器人端:ROS2 + Python (supabase-py)
- VR 端:Unity + LiveKit Unity SDK (Quest3)
- Token 服务:沿用现有实现
- 媒体:LiveKit Server(已有部署)

## 本地开发

### Supabase
```bash
cd supabase
supabase start                 # 本地起 Postgres + Studio + Auth
supabase db reset              # 重置并应用全部 migration
supabase functions deploy <fn> # 部署 Edge Function
```

### Web
```bash
cd web
npm install
cp .env.example .env.local     # 填入 NEXT_PUBLIC_SUPABASE_URL 等
npm run dev
```

### 机器人端
```bash
cd robot
# (待补充实际启动命令)
```

## 环境变量

- `NEXT_PUBLIC_SUPABASE_URL`(Web 公开)
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`(Web 公开)
- `SUPABASE_SERVICE_ROLE_KEY`(机器人端 / Edge Function,**绝不进前端**)
- `LIVEKIT_API_KEY` / `LIVEKIT_API_SECRET`(Token 服务)
- `LIVEKIT_URL`

## 测试

(待补充)

## 代码约定

- TypeScript:严格模式,禁用 `any`
- Python:black + ruff
- SQL migration 文件命名:`<timestamp>_<snake_case_name>.sql`
- Commit message:Conventional Commits(`feat:`, `fix:`, `chore:`, `refactor:` ...)
- 分支:`main`(可发布)、`feat/*`(功能)、`fix/*`(修复)
- 不要把任何 secret(LIVEKIT secret、SUPABASE service-role key)写进代码或提交到仓库

## 当前 milestone

**M1**:打通"机器人扫码 → Web 大屏接单 → 操作员手动接管"。

边界:
- token 仍走原逻辑(M2 改造)
- 机器人端使用 service-role key 写订单(M2+ 改为 per-robot 身份)
- 不做 `in_progress` 自动推进、不做 claim 超时回退(M4 处理)

## 给 Claude Code 的工作提示

- 修改 schema **必须**新建 migration 文件,不要直接改老的迁移
- 改 RLS 策略前先在 Studio 里 EXPLAIN 一下相关查询,确认没回归
- 不要把 `SUPABASE_SERVICE_ROLE_KEY` 暴露到任何前端代码或 git history
- Web 端订阅 Realtime 时,组件卸载必须 `channel.unsubscribe()`,否则连接泄漏
- LiveKit token 不要在前端硬编码 secret,必须通过 token 服务签发
- 机器人端 INSERT 订单要使用客户端生成的 UUID 作为主键,保证重试幂等
- 涉及订单状态机变更的代码,先回头看 Project knowledge 里的 `decisions.md` D5
