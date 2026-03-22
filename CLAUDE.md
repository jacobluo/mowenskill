# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mowen Note Publisher (墨问笔记发布工具) — a Python CLI tool for creating, editing, and configuring notes on 墨问 (Mowen) via its Open API. Zero third-party dependencies; uses only Python stdlib.

## Commands

```bash
# Run all tests
python3 run_tests.py

# Run a single test class
python3 -m pytest tests/test_publish_note.py::TestActionCreate -v
# Or with unittest directly:
python3 -m unittest tests.test_publish_note.TestActionCreate

# Run the script
python3 scripts/publish_note.py --action create --input note.json
python3 scripts/publish_note.py --action edit --note-id <ID> --input note.json
python3 scripts/publish_note.py --action settings --note-id <ID> --privacy public
```

No build step, no linting configuration, no CI/CD pipeline. The virtual environment is at `.venv/` but the project has zero external dependencies.

## Environment

- `MOWEN_API_KEY` — required for all API calls (or pass via `--api-key`)

## Architecture

The entire tool is a single script: `scripts/publish_note.py` (~590 lines), organized in 5 layers top-to-bottom:

1. **HTTP layer** (lines ~52-139) — `_api_request()` for JSON POST calls, `_multipart_upload()` for OSS file uploads. All requests go to `https://open.mowen.cn`.
2. **Image upload layer** (lines ~144-214) — `upload_image()` routes by source type: URL → remote 1-step upload, local path → 2-step (prepare + multipart), fileId string → passthrough.
3. **NoteAtom construction** (lines ~219-351) — `build_note_atom()` converts the simplified input JSON (paragraphs array) into the Mowen NoteAtom tree structure (`{type: "doc", content: [...]}`).
4. **Action handlers** (lines ~357-442) — `action_create()`, `action_edit()`, `action_settings()` each assemble the API payload and call `_api_request()`.
5. **CLI entry point** (lines ~464-591) — argparse-based, reads input from `--input` file or stdin, outputs result JSON to stdout, logs to stderr.

Key design decisions:
- **stdout vs stderr separation**: Result JSON goes to stdout; all progress/error logging goes to stderr via `_info()`, `_warn()`, `_die()`.
- **Rate limiting**: Built-in 1.1s delay (`RATE_LIMIT_DELAY`) between API calls to stay under the 1 req/sec limit.
- **Smart image routing**: `upload_image()` auto-detects URL vs local path vs existing fileId.

## Testing

Tests are in `tests/test_publish_note.py` (~600 lines, 10 test classes, ~60 methods). Uses `unittest` + `unittest.mock`. All network calls are mocked — no real API calls. Tests cover all content types, error paths, and CLI argument parsing.

## API Reference

Full API docs are in `references/mowen_api.md`. Key endpoints:
- `/api/open/api/v1/note/create` — create note (100/day)
- `/api/open/api/v1/note/edit` — edit API-created notes only (1000/day)
- `/api/open/api/v1/note/set` — change privacy/sharing/expiration (100/day)
- `/api/open/api/v1/upload/prepare` + `/upload/url` — image upload (200/day)

## Input JSON Format

The script accepts a JSON object with `paragraphs` (array of mixed types), optional `tags` (max 10, each ≤30 chars), `autoPublish` (bool), and `privacyType`. Paragraph entries can be: plain strings, rich-text arrays (with bold/highlight/link), or typed objects (image, heading, blockquote, bulletList, orderedList, raw).

## ClawHub 发布

### 发布流程

```bash
# 登录检查
clawhub whoami

# 发布时需排除非必要文件（.git, .venv, .claude, README.md, CLAUDE.md 等）
# clawhub publish 会打包目录下所有文件，不读 .gitignore，需用临时目录做干净发布
mkdir -p /tmp/mowenskill-clean
rsync -a --delete \
  --exclude='.git' --exclude='.venv' --exclude='.DS_Store' \
  --exclude='.claude' --exclude='.codebuddy' --exclude='__pycache__' \
  --exclude='README.md' --exclude='CLAUDE.md' --exclude='.gitignore' \
  ./ /tmp/mowenskill-clean/

clawhub publish /tmp/mowenskill-clean --slug mowenskill --version <SEMVER> --changelog "<描述>"
rm -rf /tmp/mowenskill-clean

# 验证发布结果
clawhub inspect mowenskill --files
```

### SKILL.md 规范要点（踩坑记录）

1. **`description` ≤ 200 字符** — 超过会影响触发匹配。包含关键触发词即可，不要堆砌。
2. **`name` 必须与部署时的文件夹名一致** — 提交到 ClawHub 时由 `--slug` 控制，本地目录名无所谓。
3. **声明所需环境变量** — 在 frontmatter 中用 `metadata.openclaw.requires.env` 声明，否则安全扫描会标记 `SUSPICIOUS`。
4. **不要有硬编码绝对路径** — shell 脚本中用 `$(dirname "$0")` 替代，否则安全扫描会告警。
5. **`dependencies` 字段** — 即使零依赖也建议声明 `python>=3`。
6. **用临时目录发布** — `clawhub publish` 不读 `.gitignore`，会把 `.venv/`、`.DS_Store` 等都打包进去。
7. **`--slug` 参数** — 从临时目录发布时必须指定 `--slug`，否则 slug 会取临时目录名。
