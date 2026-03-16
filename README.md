# 墨问笔记发布工具 (Mowen Note Publisher)

通过墨问 Open API 创建、编辑笔记或修改笔记设置的 Python 命令行工具。支持富文本、图片上传、标签等功能，零第三方依赖。

## 功能

- **创建笔记** — 支持纯文本、富文本（加粗/高亮/链接）、标题、引用、有序/无序列表、图片
- **编辑笔记** — 替换已通过 API 创建的笔记内容
- **修改设置** — 调整隐私级别、分享权限、过期时间

## 前置条件

- Python 3
- 墨问 API Key（从墨问小程序 → 我的 → 设置 → 开放平台 → 创建 API Key 获取）

设置环境变量：

```bash
export MOWEN_API_KEY="your_api_key"
```

## 快速开始

### 创建笔记

准备一个 JSON 文件 `note.json`：

```json
{
  "paragraphs": [
    "今天天气真好，适合出门散步",
    {"type": "image", "src": "https://example.com/photo.jpg"},
    [{"text": "重点内容", "bold": true}, " 普通文本"]
  ],
  "tags": ["日常", "心情"],
  "autoPublish": true
}
```

运行：

```bash
python3 scripts/publish_note.py --action create --input note.json
```

也可以通过 stdin 传入：

```bash
echo '{"paragraphs": ["Hello Mowen!"], "autoPublish": true}' | python3 scripts/publish_note.py --action create
```

### 编辑笔记

```bash
python3 scripts/publish_note.py --action edit --note-id <NOTE_ID> --input updated.json
```

> 仅支持编辑通过 API 创建的笔记，小程序创建的笔记无法编辑。

### 修改笔记设置

```bash
# 设为仅自己可见
python3 scripts/publish_note.py --action settings --note-id <NOTE_ID> --privacy private

# 设为公开
python3 scripts/publish_note.py --action settings --note-id <NOTE_ID> --privacy public

# 自定义规则：禁止分享 + 设置过期时间
python3 scripts/publish_note.py --action settings --note-id <NOTE_ID> --privacy rule --no-share --expire-at 1735689600
```

## 段落类型

| 类型 | 格式 | 示例 |
|------|------|------|
| 纯文本 | `"string"` | `"一段普通文本"` |
| 富文本 | `[{...}, ...]` | `[{"text": "加粗", "bold": true}, " 普通"]` |
| 标题 | `{"type": "heading", "level": N, "text": "..."}` | h1–h6 |
| 图片 | `{"type": "image", "src": "..."}` | URL、本地路径或 fileId |
| 引用 | `{"type": "blockquote", "text": "..."}` | 块引用 |
| 无序列表 | `{"type": "bulletList", "items": [...]}` | 圆点列表 |
| 有序列表 | `{"type": "orderedList", "items": [...]}` | 编号列表 |
| 原始节点 | `{"type": "raw", "node": {...}}` | 直接传入 NoteAtom 节点 |

## 图片处理

脚本自动识别图片来源并选择上传方式：

- **URL**（`http://` 或 `https://`）→ 远程上传，限制 30MB
- **本地路径** → 本地两步上传，限制 50MB，支持 gif/jpeg/jpg/png/webp
- **fileId** → 直接引用，无需上传

可选提供 `width` 和 `height` 以优化显示效果。

## 运行测试

```bash
python3 run_tests.py
```

## 项目结构

```
├── SKILL.md                     # Skill 工作流文档
├── scripts/
│   └── publish_note.py          # 主脚本（零依赖）
├── tests/
│   └── test_publish_note.py     # 单元测试
├── references/
│   └── mowen_api.md             # 墨问 API 参考文档
├── run_tests.py                 # 测试运行器
└── run_tests.sh                 # Shell 测试运行器
```

## 限制

- 全局速率限制：1 请求/秒
- 标签：最多 10 个，每个最多 30 字符（超出自动截断）
- 隐私类型：`public` 公开 / `private` 仅自己 / `rule` 自定义规则（noShare/expireAt）
- 每日配额：创建 100 次、编辑 1000 次、设置 100 次、上传 200 次

## 许可证

MIT
