# Claude Code Web

<p align="center">
  <strong>🌐 Claude Code CLI 的 Web 界面</strong><br>
  Token 流式输出 · 工具调用可视化 · Git 检查点回滚 · 多会话管理
</p>

<p align="center">
  <img src="https://img.shields.io/badge/python-3.11+-blue" alt="Python 3.11+">
  <img src="https://img.shields.io/badge/license-Apache--2.0-green" alt="Apache-2.0">
  <img src="https://img.shields.io/badge/version-1.5.0-blue" alt="v1.5.0">
</p>

---

Claude Code Web 为 [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) 提供了功能完整的浏览器界面。与 Claude 对话、通过语法高亮的 diff 视图查看工具调用、通过 Git 检查点回滚变更、上传图片和文档、管理 MCP 服务器等等——一切都在浏览器中完成。

## 功能特性

### 核心对话
- **Token 级 SSE 流式输出** — 实时查看 Claude 生成回复的过程
- **工具调用可视化** — Bash、Read、Write、Edit、Grep、Glob、WebFetch、WebSearch 及 MCP 工具以 diff 视图和语法高亮的方式渲染
- **权限模式控制** — default、acceptEdits、bypassPermissions、plan
- **模型选择** — 选择 CLI 支持的任何 Claude 模型
- **自定义系统提示** — 每条消息可使用 `--append-system-prompt`
- **工具白名单/黑名单** — 每次对话可限制 Claude 使用哪些工具
- **停止生成** — 随时中止正在运行的请求

### 文件与文档上传
- **图片** — PNG、JPG、GIF、WebP、BMP（以 base64 vision 输入发送）
- **文档** — PDF（含 pdfplumber 回退）、DOCX（保留段落/表格顺序、页眉页脚）、XLSX/XLSM/XLS（所有工作表）、PPTX（幻灯片 + 备注 + 表格）、HTML
- **文本文件** — 自动检测编码（UTF-8、UTF-16、GB18030），拒绝二进制文件
- **上传文件保留 30 天**，自动清理

### 会话管理
- **置顶、归档、标签** — 整理会话
- **全文搜索** — 搜索会话标题和对话内容
- **Fork 分支** — 从任意对话轮次分支，携带之前的上下文发起新消息
- **行内编辑** — 重写任意历史用户消息并从该点继续
- **会话压缩** — 使用 Claude Haiku 将长对话历史总结为摘要前缀，保留最后 N 轮
- **Markdown 导出** — 将会话下载为 .md 文件
- **AI 标题建议** — 自动生成中文会话标题
- **建议追问** — 每个会话生成 3 个上下文相关的后续问题

### Git 集成
- **自动检查点** — 每轮对话前自动创建 git stash
- **一键回滚** — 将工作区恢复到任意轮次的检查点
- **页头 Git 状态** — 显示当前分支和变更文件数

### MCP 服务器管理
- **三种作用域** — User（`~/.claude.json`）、Local（按项目存储在 `~/.claude.json`）、Project（`.mcp.json`）
- **完整 CRUD** — 从界面中添加、编辑、删除 MCP 服务器
- **启用/禁用** — 无需删除配置即可切换服务器状态
- **凭证遮盖** — token、密钥等敏感信息在界面中被遮盖

### 持久化记忆
- **三种作用域** — 全局、项目（按目录）、会话
- **自动注入** — 已启用的记忆作为系统提示上下文自动附加
- **CRUD 界面** — 在设置面板中管理记忆

### 用量与分析
- **每会话 Token 追踪** — 输入、输出、缓存读取、缓存创建
- **费用估算** — 每轮和每会话的美元费用
- **仪表板统计** — 总费用、总轮次、每日明细、工具使用排行

### 开发者工具
- **代码执行沙箱** — 运行 Python、JavaScript、Bash 代码片段（30 秒超时）
- **URL 抓取器** — 从任意公开 URL 抓取并提取文本（SSRF 防护）
- **文件浏览器** — 通过 `git ls-files` 浏览项目文件（快速），支持搜索
- **项目扫描器** — 从配置文件自动检测项目类型，显示目录树
- **自定义斜杠提示** — 保存并复用提示模板，通过斜杠触发

### 认证与安全
- **JWT 认证**（可选）— HMAC-SHA256，24 小时过期，PBKDF2 密钥派生
- **SSRF 防护** — URL 抓取器阻止私有/内部 IP
- **路径遍历防护** — 文档内容接口锁定上传目录
- **凭证遮盖** — MCP 密钥不会以明文形式离开服务器

### 界面体验
- **暗色模式** — 跟随系统偏好
- **移动端适配** — 支持手机和平板
- **多会话标签页** — 在活跃对话之间切换
- **液态玻璃设计** — 精致现代的视觉风格

## 快速开始

### 环境要求

- **Python 3.11+**
- **[Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)** 已安装并认证：
  ```bash
  npm install -g @anthropic-ai/claude-code
  claude  # 首次登录
  ```

### 安装

```bash
pip install claude-web-ui
```

### 运行

```bash
claude-web
# → http://127.0.0.1:8765
```

### 启用认证

```bash
claude-web --auth-user admin --auth-password yourpassword --host 0.0.0.0
```

或通过环境变量：

```bash
export CLAUDE_WEB_USER=admin
export CLAUDE_WEB_PASSWORD=yourpassword
claude-web --host 0.0.0.0
```

> **安全提示：** 绑定 `0.0.0.0` 时，务必启用认证，并仅在可信网络中暴露（Tailscale、VPN、SSH 隧道）。

## 命令行参考

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `--port`, `-p` | `8765` | 监听端口（也可通过 `PORT` 环境变量设置） |
| `--host` | `127.0.0.1` | 绑定地址 |
| `--auth-user` | （环境变量：`CLAUDE_WEB_USER`） | JWT 认证用户名 |
| `--auth-password` | （环境变量：`CLAUDE_WEB_PASSWORD`） | JWT 认证密码 |
| `--version`, `-v` | | 显示版本并退出 |
| `--open` | | 启动时自动打开浏览器 |
| `--skip-cli-check` | | 跳过 Claude CLI 可用性检查 |

## 部署为 Systemd 服务

```ini
# /etc/systemd/system/claude-web.service
[Unit]
Description=Claude Code Web UI
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=your-user
ExecStart=/home/your-user/.local/bin/claude-web --host 0.0.0.0 --port 8765 --auth-user admin --auth-password yourpassword
Restart=always
RestartSec=5
Environment=CLAUDE_WEB_DATA_DIR=/home/your-user/claude-web-data

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now claude-web
```

## 数据存储

所有持久化数据存放在工作目录下（如果设置了 `CLAUDE_WEB_DATA_DIR` 则使用该路径）：

```
.
├── claude-web.db          # SQLite — 会话、提示、记忆、用量
├── history/
│   ├── <session-id>.jsonl # 每个会话的事件日志
│   └── *.before-compact-* # 压缩备份（保留最近 3 个，7 天）
└── uploads/               # 上传文件（30 天后自动清理）
```

## API 接口

<details>
<summary>点击展开完整 API 参考</summary>

### 对话
| 方法 | 路径 | 说明 |
|------|------|------|
| `POST` | `/api/chat` | 发送消息，返回 SSE 流 |
| `POST` | `/api/chat/stop/{session_id}` | 停止正在运行的生成 |

### 会话
| 方法 | 路径 | 说明 |
|------|------|------|
| `GET` | `/api/sessions` | 列出会话（支持 `?q=`、`?archived=`、`?tag=`） |
| `GET` | `/api/sessions/search` | 快速搜索（`?q=`、`?limit=`） |
| `GET` | `/api/sessions/{id}` | 获取会话及事件 |
| `PATCH` | `/api/sessions/{id}` | 更新标题、置顶、归档、标签 |
| `DELETE` | `/api/sessions/{id}` | 删除会话及所有数据 |
| `POST` | `/api/sessions/{id}/clear` | 清空对话历史 |
| `POST` | `/api/sessions/{id}/compact` | 压缩历史（`?keep_last=2`） |
| `POST` | `/api/sessions/{id}/suggest-title` | AI 生成标题 |
| `POST` | `/api/sessions/{id}/prepare-fork` | 准备从某轮分支 |
| `POST` | `/api/sessions/{id}/prepare-inline-edit` | 准备行内编辑 |
| `POST` | `/api/sessions/{id}/restore-checkpoint` | Git 回滚到某轮 |
| `GET` | `/api/sessions/{id}/export` | 下载为 Markdown |
| `GET` | `/api/sessions/{id}/usage` | Token/费用明细 |
| `GET` | `/api/sessions/{id}/mention` | 获取会话上下文用于 @ 提及 |

### 上传
| 方法 | 路径 | 说明 |
|------|------|------|
| `POST` | `/api/upload` | 上传图片（最大 20MB） |
| `POST` | `/api/upload-doc` | 上传文档（最大 30MB，自动提取文本） |
| `GET` | `/api/doc-content` | 读取已提取的文档文本 |

### MCP 服务器
| 方法 | 路径 | 说明 |
|------|------|------|
| `GET` | `/api/mcp/servers` | 列出所有 MCP 服务器 |
| `POST` | `/api/mcp/servers/{name}` | 添加服务器（`?scope=local`、`?cwd=`） |
| `PATCH` | `/api/mcp/servers/{name}` | 更新服务器配置 |
| `DELETE` | `/api/mcp/servers/{name}` | 删除服务器 |

### 记忆
| 方法 | 路径 | 说明 |
|------|------|------|
| `GET` | `/api/memories` | 列出记忆（`?scope=`、`?q=`） |
| `GET` | `/api/memories/active` | 获取当前 cwd+会话的活跃记忆 |
| `POST` | `/api/memories` | 创建记忆 |
| `PUT` | `/api/memories/{id}` | 更新记忆 |
| `DELETE` | `/api/memories/{id}` | 删除记忆 |

### 提示模板
| 方法 | 路径 | 说明 |
|------|------|------|
| `GET` | `/api/prompts` | 列出自定义提示 |
| `POST` | `/api/prompts` | 创建提示（可选斜杠触发器） |
| `PUT` | `/api/prompts/{id}` | 更新提示 |
| `DELETE` | `/api/prompts/{id}` | 删除提示 |

### 工具与实用功能
| 方法 | 路径 | 说明 |
|------|------|------|
| `POST` | `/api/exec-code` | 运行 Python/JS/Bash 代码片段 |
| `POST` | `/api/fetch-url` | 抓取并提取网页文本 |
| `GET` | `/api/files` | 浏览项目文件（`?cwd=`、`?q=`） |
| `GET` | `/api/git` | Git 分支和变更状态 |
| `GET` | `/api/stats` | 用量仪表板 |
| `GET` | `/api/cwds` | 最近工作目录 |
| `GET` | `/api/tags` | 所有标签及计数 |
| `GET` | `/api/version` | 服务器版本 |
| `GET` | `/changelog.json` | 更新日志 |

### 认证
| 方法 | 路径 | 说明 |
|------|------|------|
| `POST` | `/api/auth/login` | 登录，返回 JWT |
| `GET` | `/api/auth/check` | 检查是否启用了认证 |

</details>

## 环境变量配置

| 变量 | 说明 |
|------|------|
| `CLAUDE_WEB_USER` | 认证用户名 |
| `CLAUDE_WEB_PASSWORD` | 认证密码 |
| `CLAUDE_WEB_DATA_DIR` | 数据目录（默认：`$PWD`） |
| `PORT` | 监听端口（默认：`8765`） |

## 架构

```
claude-web (FastAPI + Uvicorn)
├── /api/chat  ── SSE 流 ──→  claude CLI 子进程 (stream-json)
├── /api/*     ── REST API ──→  SQLite + JSONL 历史 + ~/.claude.json
└── /           ── 静态 SPA (index.html, vanilla JS)
```

服务器为每个对话请求启动一个 `claude` CLI 子进程，将 `stream-json` 输出以 Server-Sent Events 方式推送到浏览器，并将所有事件持久化到 JSONL 文件中。认证通过中间件实现 JWT 验证；不传 `--auth-*` 参数时认证默认关闭。

## 开发

```bash
git clone https://github.com/ItsPiura/claude-web.git
cd claude-web
pip install -e .
claude-web --port 8765
```

前端是单文件 SPA，位于 `claude_web/static/index.html`。后端是单个 FastAPI 模块，位于 `claude_web/server.py`。

## 依赖要求

- Python 3.11+
- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- 依赖（自动安装）：FastAPI、uvicorn、pypdf、python-docx、openpyxl、xlrd、python-pptx、python-multipart

## 许可证

Apache-2.0 © ItsPiura
