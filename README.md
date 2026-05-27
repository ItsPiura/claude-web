# Claude Code Web

Web UI for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI with token streaming, tool visualization, checkpoint rollback, and multi-session management.

## Features

- Token-level streaming output
- Tool call visualization with diff views
- Image & document upload (PDF, DOCX, XLSX, PPTX, etc.)
- Git checkpoint rollback
- Session pinning, archiving, tagging, search, export
- MCP server management
- JWT-based authentication (optional)
- Dark mode & mobile responsive

## Quick Start

```bash
pip install claude-web-ui
claude-web
```

Open http://127.0.0.1:8765

## Authentication (optional)

```bash
claude-web --auth-user admin --auth-password mypassword --host 0.0.0.0
```

Or via environment variables:

```bash
export CLAUDE_WEB_USER=admin
export CLAUDE_WEB_PASSWORD=mypassword
claude-web --host 0.0.0.0
```

## CLI Options

| Flag | Default | Description |
|------|---------|-------------|
| `--port, -p` | 8765 | Port to listen on |
| `--host` | 127.0.0.1 | Host to bind |
| `--auth-user` | (env: `CLAUDE_WEB_USER`) | Auth username |
| `--auth-password` | (env: `CLAUDE_WEB_PASSWORD`) | Auth password |
| `--version, -v` | | Show version |
| `--open` | | Auto-open browser |

## Requirements

- Python 3.11+
- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
