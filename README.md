# Claude Code Web

<p align="center">
  <strong>🌐 Web UI for Claude Code CLI</strong><br>
  Token streaming · Tool visualization · Git checkpoint rollback · Multi-session management
</p>

<p align="center">
  <img src="https://img.shields.io/badge/python-3.11+-blue" alt="Python 3.11+">
  <img src="https://img.shields.io/badge/license-Apache--2.0-green" alt="Apache-2.0">
  <img src="https://img.shields.io/badge/version-1.5.0-blue" alt="v1.5.0">
</p>

---

Claude Code Web wraps the [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) with a full-featured browser UI. Chat with Claude, visualize tool calls with syntax-highlighted diffs, rollback changes via git checkpoints, upload images and documents, manage MCP servers, and more — all from your browser.

## Features

### Core Chat
- **Token-level SSE streaming** — see Claude's response as it's generated
- **Tool call visualization** — Bash, Read, Write, Edit, Grep, Glob, WebFetch, WebSearch, and MCP tools rendered with diff views and syntax highlighting
- **Permission mode control** — default, acceptEdits, bypassPermissions, plan
- **Model selection** — choose any Claude model your CLI supports
- **Custom system prompts** — per-message `--append-system-prompt`
- **Allowed/disallowed tools** — restrict which tools Claude can use per turn
- **Stop generation** — cancel a running request mid-stream

### File & Document Upload
- **Images** — PNG, JPG, GIF, WebP, BMP (sent as base64 vision input)
- **Documents** — PDF (with pdfplumber fallback), DOCX (preserves paragraph/table order, headers/footers), XLSX/XLSM/XLS (all sheets), PPTX (slides + notes + tables), HTML
- **Text files** — auto-detects encoding (UTF-8, UTF-16, GB18030), rejects binaries
- **30-day upload retention** with automatic pruning

### Session Management
- **Pin, archive, tag** sessions for organization
- **Full-text search** across session titles and conversation content
- **Fork** — branch from any turn with a new message, carrying prior context
- **Inline edit** — rewrite any past user message and continue from that point
- **Session compaction** — summarize long history into a summary prefix using Claude Haiku, keeping the last N turns intact
- **Markdown export** — download any session as a .md file
- **AI title suggestions** — auto-generate Chinese session titles
- **Suggested follow-ups** — 3 contextual follow-up questions per session

### Git Integration
- **Automatic checkpoints** — git stash created before each turn
- **One-click rollback** — restore working tree to any turn's checkpoint
- **Git status in header** — current branch and dirty file count

### MCP Server Management
- **Three scopes** — User (`~/.claude.json`), Local (per-project in `~/.claude.json`), Project (`.mcp.json`)
- **Full CRUD** — add, edit, delete MCP servers from the UI
- **Enable/disable** — toggle servers without removing config
- **Credential masking** — tokens, keys, secrets are masked in the UI

### Persistent Memory
- **Three scopes** — Global, Project (per-directory), Session
- **Auto-injected** — enabled memories are appended as system prompt context
- **CRUD interface** — manage memories from the settings panel

### Usage & Analytics
- **Per-session token tracking** — input, output, cache read, cache creation
- **Cost estimation** — USD cost per turn and per session
- **Dashboard stats** — total cost, total turns, daily breakdown, top tools

### Developer Tools
- **Code execution sandbox** — run Python, JavaScript, Bash snippets (30s timeout)
- **URL fetcher** — fetch and extract text from any public URL (SSRF-protected)
- **File browser** — browse project files via `git ls-files` (fast) with search
- **Project scanner** — auto-detect project type from config files, show directory tree
- **Custom slash prompts** — save and reuse prompt templates with slash triggers

### Auth & Security
- **JWT authentication** (optional) — HMAC-SHA256, 24h expiry, PBKDF2 key derivation
- **SSRF protection** — URL fetcher blocks private/internal IPs
- **Path traversal protection** — doc content endpoint locked to uploads directory
- **Credential masking** — MCP secrets never leave the server in plaintext

### UI/UX
- **Dark mode** — follows system preference
- **Mobile responsive** — works on phones and tablets
- **Multi-session tabs** — switch between active conversations
- **Liquid glass design** — polished, modern aesthetic

## Quick Start

### Prerequisites

- **Python 3.11+**
- **[Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)** installed and authenticated:
  ```bash
  npm install -g @anthropic-ai/claude-code
  claude  # one-time login
  ```

### Install

```bash
pip install claude-web-ui
```

### Run

```bash
claude-web
# → http://127.0.0.1:8765
```

### With Authentication

```bash
claude-web --auth-user admin --auth-password yourpassword --host 0.0.0.0
```

Or via environment variables:

```bash
export CLAUDE_WEB_USER=admin
export CLAUDE_WEB_PASSWORD=yourpassword
claude-web --host 0.0.0.0
```

> **Security note:** When binding to `0.0.0.0`, always enable authentication and only expose on trusted networks (Tailscale, VPN, SSH tunnel).

## CLI Reference

| Flag | Default | Description |
|------|---------|-------------|
| `--port`, `-p` | `8765` | Port to listen on (also `PORT` env var) |
| `--host` | `127.0.0.1` | Host to bind |
| `--auth-user` | (env: `CLAUDE_WEB_USER`) | Username for JWT auth |
| `--auth-password` | (env: `CLAUDE_WEB_PASSWORD`) | Password for JWT auth |
| `--version`, `-v` | | Show version and exit |
| `--open` | | Auto-open browser on start |
| `--skip-cli-check` | | Skip Claude CLI availability check |

## Deploy as a Systemd Service

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

## Data Storage

All persistent data lives under the working directory (or `CLAUDE_WEB_DATA_DIR` if set):

```
.
├── claude-web.db          # SQLite — sessions, prompts, memories, usage
├── history/
│   ├── <session-id>.jsonl # Per-session event logs
│   └── *.before-compact-* # Compaction backups (kept 3 latest, 7 days)
└── uploads/               # Uploaded files (pruned after 30 days)
```

## API Endpoints

<details>
<summary>Click to expand full API reference</summary>

### Chat
| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/chat` | Send message, returns SSE stream |
| `POST` | `/api/chat/stop/{session_id}` | Stop running generation |

### Sessions
| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/sessions` | List sessions (supports `?q=`, `?archived=`, `?tag=`) |
| `GET` | `/api/sessions/search` | Quick search (`?q=`, `?limit=`) |
| `GET` | `/api/sessions/{id}` | Get session with events |
| `PATCH` | `/api/sessions/{id}` | Update title, pin, archive, tags |
| `DELETE` | `/api/sessions/{id}` | Delete session and all data |
| `POST` | `/api/sessions/{id}/clear` | Clear conversation history |
| `POST` | `/api/sessions/{id}/compact` | Summarize history (`?keep_last=2`) |
| `POST` | `/api/sessions/{id}/suggest-title` | AI-generated title |
| `POST` | `/api/sessions/{id}/prepare-fork` | Prepare fork from turn |
| `POST` | `/api/sessions/{id}/prepare-inline-edit` | Prepare inline edit |
| `POST` | `/api/sessions/{id}/restore-checkpoint` | Git rollback to turn |
| `GET` | `/api/sessions/{id}/export` | Download as Markdown |
| `GET` | `/api/sessions/{id}/usage` | Token/cost breakdown |
| `GET` | `/api/sessions/{id}/mention` | Get session context for @-mentions |

### Upload
| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/upload` | Upload image (max 20MB) |
| `POST` | `/api/upload-doc` | Upload document (max 30MB, auto-extract) |
| `GET` | `/api/doc-content` | Read back extracted doc text |

### MCP Servers
| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/mcp/servers` | List all MCP servers |
| `POST` | `/api/mcp/servers/{name}` | Add server (`?scope=local`, `?cwd=`) |
| `PATCH` | `/api/mcp/servers/{name}` | Update server config |
| `DELETE` | `/api/mcp/servers/{name}` | Remove server |

### Memory
| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/memories` | List memories (`?scope=`, `?q=`) |
| `GET` | `/api/memories/active` | Active memories for cwd+session |
| `POST` | `/api/memories` | Create memory |
| `PUT` | `/api/memories/{id}` | Update memory |
| `DELETE` | `/api/memories/{id}` | Delete memory |

### Prompts
| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/prompts` | List custom prompts |
| `POST` | `/api/prompts` | Create prompt (with optional slash trigger) |
| `PUT` | `/api/prompts/{id}` | Update prompt |
| `DELETE` | `/api/prompts/{id}` | Delete prompt |

### Tools & Utilities
| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/exec-code` | Run Python/JS/Bash snippet |
| `POST` | `/api/fetch-url` | Fetch and extract webpage text |
| `GET` | `/api/files` | Browse project files (`?cwd=`, `?q=`) |
| `GET` | `/api/git` | Git branch and dirty status |
| `GET` | `/api/stats` | Usage dashboard |
| `GET` | `/api/cwds` | Recent working directories |
| `GET` | `/api/tags` | All tags with counts |
| `GET` | `/api/version` | Server version |
| `GET` | `/changelog.json` | Changelog data |

### Auth
| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/auth/login` | Login, returns JWT |
| `GET` | `/api/auth/check` | Check if auth is enabled |

</details>

## Configuration via Environment Variables

| Variable | Description |
|----------|-------------|
| `CLAUDE_WEB_USER` | Auth username |
| `CLAUDE_WEB_PASSWORD` | Auth password |
| `CLAUDE_WEB_DATA_DIR` | Data directory (default: `$PWD`) |
| `PORT` | Listen port (default: `8765`) |

## Architecture

```
claude-web (FastAPI + Uvicorn)
├── /api/chat  ── SSE stream ──→  claude CLI subprocess (stream-json)
├── /api/*     ── REST API   ──→  SQLite + JSONL history + ~/.claude.json
└── /           ── Static SPA (index.html, vanilla JS)
```

The server spawns a `claude` CLI subprocess per chat request, pipes `stream-json` output back to the browser as Server-Sent Events, and persists all events to JSONL files. Auth is JWT-based via middleware; without `--auth-*` flags it's disabled.

## Development

```bash
git clone https://github.com/ItsPiura/claude-web.git
cd claude-web
pip install -e .
claude-web --port 8765
```

The frontend is a single-file SPA at `claude_web/static/index.html`. The backend is a single FastAPI module at `claude_web/server.py`.

## Requirements

- Python 3.11+
- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- Dependencies (auto-installed): FastAPI, uvicorn, pypdf, python-docx, openpyxl, xlrd, python-pptx, python-multipart

## License

Apache-2.0 © ItsPiura
