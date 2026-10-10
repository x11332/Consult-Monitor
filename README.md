这是一个可以实时获取信息的咨讯搜索工具

获取skill请执行命令：

npx skills add https://github.com/x11332/Consult-Monitor --skill consult-monitor

# Consult Monitor

Consult Monitor是一个多来源热点监控项目。添加关键词后，系统会定时搜索国内外公开来源，分析并保存相关内容；你也可以在仪表盘筛选热点、查看通知，或手动触发检查。

仓库还包含一组可独立运行的 Python 搜索脚本，适用于不启动 Web 应用时的临时搜索和报告生成。

## 功能

- 管理监控关键词，并启用或停用关键词。
- 从 Twitter/X、Bing、Hacker News、搜狗、Bilibili 和微博收集内容。
- 使用 AI 扩展关键词、判断内容相关性并生成摘要（配置 OpenRouter 后启用）。
- 查看热点统计、来源、相关性和重要程度；按关键词、来源、时间等条件筛选与排序。
- 通过 WebSocket 实时接收新热点和通知。
- 每 30 分钟自动检查一次，也可以从界面手动触发。
- 可选发送热点邮件通知。

## 技术栈

| 部分 | 技术 |
| --- | --- |
| 前端 | React、TypeScript、Vite、Tailwind CSS |
| 后端 | Node.js、Express、TypeScript、Socket.IO |
| 数据库 | SQLite、Prisma |
| 独立搜索脚本 | Python、Requests、Beautiful Soup |

## 环境要求

- Node.js 22 LTS（建议使用当前受支持的 LTS 版本）
- npm
- Python 3（仅运行独立搜索脚本时需要）

SQLite 使用本地数据库文件，无需单独安装或启动数据库服务。

## 本地运行 Web 应用

### 1. 配置后端

在仓库根目录打开终端：

```powershell
cd server
Copy-Item .env.example .env
npm install
```

根据需要编辑 `server/.env`：

| 变量 | 用途 | 是否必需 |
| --- | --- | --- |
| `DATABASE_URL` | SQLite 数据库地址，示例值为 `file:./dev.db` | 否；示例配置可直接使用 |
| `PORT` | 后端端口，默认为 `3001` | 否 |
| `CLIENT_URL` | 前端来源地址，用于 Socket.IO CORS，默认为 `http://localhost:5173` | 否 |
| `OPENROUTER_API_KEY` | OpenRouter API 密钥，用于 AI 关键词扩展和内容分析 | 否 |
| `TWITTER_API_KEY` | twitterapi.io API 密钥，用于搜索 Twitter/X | 否；不配置时跳过该来源 |
| `SMTP_HOST`、`SMTP_PORT`、`SMTP_SECURE`、`SMTP_USER`、`SMTP_PASS` | SMTP 邮件服务器配置 | 否 |
| `NOTIFY_EMAIL` | 接收热点邮件的地址 | 否 |

生成 Prisma Client 并创建或同步本地数据库：

```powershell
npm run db:generate
npm run db:push
```

启动后端开发服务器：

```powershell
npm run dev
```

### 2. 启动前端

保持后端运行，在另一个终端中执行：

```powershell
cd client
npm install
npm run dev
```

打开 Vite 输出的本地地址（通常为 <http://localhost:5173>）。开发服务器会将 `/api` 和 `/socket.io` 请求代理到后端。

### 3. 开始监控

在界面添加监控关键词。后端会每 30 分钟自动检查已启用的关键词；也可以在应用中手动触发热点检查。要使用 AI 分析、Twitter/X 搜索或邮件通知，请先在 `server/.env` 中填写对应配置。

## 构建、测试与数据库工具

在各自目录执行对应命令：

| 目录 | 命令 | 说明 |
| --- | --- | --- |
| `server` | `npm run build` | 编译后端 TypeScript |
| `server` | `npm test` | 运行后端 Vitest 测试 |
| `server` | `npm start` | 运行已编译的后端（先执行构建） |
| `server` | `npm run db:studio` | 打开 Prisma Studio 查看数据库 |
| `server` | `npm run db:push` | 将 Prisma schema 同步到 SQLite |
| `client` | `npm run build` | 类型检查并构建前端到 `client/dist` |
| `client` | `npm run lint` | 运行 ESLint |
| `client` | `npm run preview` | 本地预览前端构建结果 |

`npm run db:push` 适用于本地开发和快速初始化；修改数据库模型时，如果需要生成并保留迁移记录，可使用 `npm run db:migrate`。

## 独立 Python 搜索脚本

这些脚本位于 `skills/consult-monitor/scripts/`，可以不启动 Web 应用单独使用。安装依赖：

```powershell
python -m pip install -r skills/consult-monitor/scripts/requirements.txt
```

示例：

```powershell
python skills/consult-monitor/scripts/search_web.py "AI programming" --sources bing,hackernews,duckduckgo
python skills/consult-monitor/scripts/search_china.py "AI编程" --sources sogou,bilibili,weibo
```

`search_twitter.py` 需要设置 `TWITTER_API_KEY`。`generate_report.py` 可将搜索结果 JSON 转换为 Markdown 报告。更多参数和分析指南见 [技能说明](skills/consult-monitor/SKILL.md)、[数据源参考](skills/consult-monitor/references/search-sources.md) 和 [分析指南](skills/consult-monitor/references/analysis-guide.md)。

## 项目结构

```text
client/                         React 前端
server/                         Express 后端、Prisma schema 和数据库迁移
skills/consult-monitor/         独立的 AI 搜索技能与 Python 脚本
```
