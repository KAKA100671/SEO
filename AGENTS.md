# AGENTS.md · SEO（ai-seo-audit）

> 用途：Next.js 网页应用——输入一个 URL，按 30+ 条规则给出「AI 回答引擎可读性」0–100 分（可抓取性、内容分块、结构化数据、E-E-A-T、站外引用、治理六类），并生成可粘贴给编码 agent 的修复提示词｜状态：来自上游开源项目 ai-seo-audit（MIT）的副本，提交历史全部来自上游作者（2026-05-11 至 05-13），本仓暂无独立改动，**未开发维护**｜远端：`KAKA100671/SEO`（公开）

## 是什么

一个标准 Next.js 16（App Router、React 19、Tailwind v4）应用。页面内容的抓取与 Markdown/HTML 提取全部通过第三方服务 context.dev 的 SDK 完成（应用本身不直接访问被审计站点）；审计结果可选用 Turso（libSQL）做缓存。详细架构图、规则编写方式与 API 示例见 `README.md`。

## 结构

| 路径 | 是什么 |
|---|---|
| `app/` | 页面与接口：`page.tsx` 首页、`audit/[domain]/` 结果页、`api/audit/route.ts` JSON 接口（GET/POST）、`robots.ts`、`sitemap.ts` |
| `components/` | 审计表单与结果视图 |
| `lib/rules.ts` | **全部审计规则的唯一出处**（增删/调权重只改这里；评分、界面、修复提示词都由它派生） |
| `lib/audit.ts`、`lib/perform-audit.ts` | 审计编排（抓取 → 解析 → 打分）、缓存与错误封装 |
| `lib/agent-fix-prompt.ts` | 把失败的规则整理成给编码 agent 的提示词 |
| `lib/turso.ts` | 可选缓存（按域名与 schema 版本做 key） |
| `lib/url.ts`、`lib/brand*.ts`、`lib/audit-types.ts` | URL 规范化、品牌常量、类型 |
| `public/` | 图标与站点清单 |

## 怎么跑

需要 Node 20+。

```bash
npm install
cp .env.example .env     # 然后填写环境变量
npm run dev              # http://localhost:3000
npm run build && npm run start
```

环境变量（只写名字，值自行申请，不要提交）：`CONTEXT_DEV_API_KEY`（必填，context.dev 的 API Key）、`NEXT_PUBLIC_SITE_URL`（必填，站点规范地址，用于 OG 标签与 sitemap）、`aiseo_TURSO_DATABASE_URL` 与 `aiseo_TURSO_AUTH_TOKEN`（可选，缓存；也认不带前缀的 `TURSO_*`）。

## 怎么验

仓库**没有单元测试**。改动后跑三条（需先 `npm install`）：

```bash
npm run lint
npm run typecheck
npm run build
```

再手动请求一次 `GET /api/audit?domain=example.com`（需要有效的 `CONTEXT_DEV_API_KEY`，会产生第三方调用额度消耗）看返回结构。本次接手审计**未实跑**以上命令（未安装依赖，且接口调用需要密钥）。

## 怎么发布

上游推荐 Vercel（+ Turso Marketplace）：导入仓库、在项目环境变量里配置上面的变量、部署。也可自托管（标准 Next.js，Node 20+）。本仓没有 CI 配置文件。

## 红线

- 公开仓库：只放代码与通用文档，不放任何 Key、令牌、账号、内部系统信息、个人或公司内部路径。`.env` 已被 `.gitignore` 忽略，只提交 `.env.example`（值必须保持为空）。
- 保留 MIT 许可文件与上游署名；不要冒充上游项目或其品牌发布。
- 修改规则只动 `lib/rules.ts`，并在提交说明里写明依据（上游贡献约定要求为启发式规则注明来源）。

## 已知坑与待核实

- **不设 `NEXT_PUBLIC_SITE_URL` 就部署，规范地址、OG、sitemap 会回落到上游作者的域名**（代码里写死了默认值），自行部署务必配置。
- 缓存键含 schema 版本（`lib/perform-audit.ts` 里的 `CACHE_SCHEMA_VERSION`）；改了审计结果结构或规则打分后要同步抬高它，否则会读到旧缓存。
- 依赖版本较新（Next 16、TypeScript 6、React 19），升级前先看 `package-lock.json` 并跑上面三条。
- 未知：该副本的用途、是否计划二次开发、是否已部署（本仓没有说明），待所有者确认。
- 与上游的同步方式（是否还要合并上游更新）未约定，待核实。

## 改动记录（新的在上）

| 日期 | 谁 | 改了什么 | 验收证据 |
|---|---|---|---|
| 2026-10-21 | Claude（接手审计） | 新增本 AGENTS.md（只文档，不改代码） | 未实跑 lint/typecheck/build（依赖未装）；暂存区 gitleaks 扫描；人工检查文中无内部信息 |
