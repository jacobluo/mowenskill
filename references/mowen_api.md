# 墨问 Open API 参考文档

## 概述

- **Base URL**: `https://open.mowen.cn`
- **认证方式**: Bearer Token（API-KEY）
- **请求头**: `Authorization: Bearer {API-KEY}`, `Content-Type: application/json`
- **全局限频**: 所有 API 均限频 1次/秒

---

## 1. 笔记创建

### POST `/api/open/api/v1/note/create`

创建一篇新笔记。

| 属性 | 说明 |
|------|------|
| 限频 | 1次/秒 |
| 配额 | 100次/天 |

### 请求体

```json
{
  "noteContent": {
    "noteAtom": { ... }
  },
  "noteSetting": {
    "tags": ["标签1", "标签2"],
    "autoPublish": true,
    "privacyType": 1
  }
}
```

#### 字段说明

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `noteContent.noteAtom` | object | ✅ | NoteAtom 树结构，详见 [NoteAtom 结构](#3-noteatom-结构定义) |
| `noteSetting.tags` | string[] | ❌ | 标签列表，最多 10 个，每个不超过 30 字符 |
| `noteSetting.autoPublish` | boolean | ❌ | 是否自动发布，默认 `false`（草稿） |
| `noteSetting.privacyType` | integer | ❌ | 隐私类型：`1`=公开, `2`=仅自己可见, `3`=部分可见, `4`=不给谁看 |

### 成功响应

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "noteId": "note_abc123"
  }
}
```

---

## 2. 笔记编辑

### POST `/api/open/api/v1/note/edit`

编辑已有笔记的内容。

> ⚠️ **重要限制**: 仅支持编辑通过 API 创建的笔记，不支持小程序端创建的笔记。

| 属性 | 说明 |
|------|------|
| 限频 | 1次/秒 |
| 配额 | 1000次/天 |

### 请求体

```json
{
  "noteId": "note_abc123",
  "noteContent": {
    "noteAtom": { ... }
  }
}
```

#### 字段说明

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `noteId` | string | ✅ | 笔记 ID（创建时返回） |
| `noteContent.noteAtom` | object | ✅ | 新的 NoteAtom 树结构，会完全替换原有内容 |

### 成功响应

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "noteId": "note_abc123"
  }
}
```

---

## 3. 笔记设置

### POST `/api/open/api/v1/note/settings`

修改笔记的隐私和分享设置。

| 属性 | 说明 |
|------|------|
| 限频 | 1次/秒 |
| 配额 | 100次/天 |

### 请求体

```json
{
  "noteId": "note_abc123",
  "noteSetting": {
    "privacyType": 2,
    "forbidShare": true,
    "expireTime": 1700000000
  }
}
```

#### 字段说明

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `noteId` | string | ✅ | 笔记 ID |
| `noteSetting.privacyType` | integer | ❌ | `1`=公开, `2`=仅自己可见, `3`=部分可见, `4`=不给谁看 |
| `noteSetting.forbidShare` | boolean | ❌ | 是否禁止分享 |
| `noteSetting.expireTime` | integer | ❌ | 过期时间，Unix 时间戳（秒） |

### 成功响应

```json
{
  "code": 0,
  "msg": "success",
  "data": {}
}
```

---

## 4. NoteAtom 结构定义

NoteAtom 是墨问笔记的富文本内容结构，采用树状嵌套形式。

### 根节点

```json
{
  "type": "doc",
  "content": [ ... ]
}
```

根节点 `type` **必须**为 `"doc"`，`content` 为子节点数组。

### 节点类型

#### 段落 (paragraph)

```json
{
  "type": "paragraph",
  "content": [
    {
      "type": "text",
      "text": "这是一段文字"
    }
  ]
}
```

#### 文本 (text)

文本节点是叶子节点，可以带有格式标记（marks）。

```json
{
  "type": "text",
  "text": "加粗高亮文字",
  "marks": [
    { "type": "bold" },
    { "type": "highlight" }
  ]
}
```

**支持的 marks 类型：**

| Mark 类型 | 说明 | 附加属性 |
|-----------|------|----------|
| `bold` | 加粗 | 无 |
| `highlight` | 高亮 | 无 |
| `link` | 超链接 | `attrs: { href: "https://..." }` |

#### 链接文本示例

```json
{
  "type": "text",
  "text": "点击这里",
  "marks": [
    {
      "type": "link",
      "attrs": {
        "href": "https://example.com"
      }
    }
  ]
}
```

#### 图片 (noteImage)

```json
{
  "type": "noteImage",
  "attrs": {
    "uuid": "file_id_from_upload",
    "width": 1080,
    "height": 720,
    "ratio": 1.5
  }
}
```

| 属性 | 类型 | 说明 |
|------|------|------|
| `uuid` | string | 图片文件 ID，来自上传 API 返回的 `fileId` |
| `width` | integer | 图片宽度（px） |
| `height` | integer | 图片高度（px） |
| `ratio` | number | 宽高比 `width / height` |

> 图片尺寸信息可选但推荐提供，用于前端渲染占位。

#### 引用块 (blockquote)

```json
{
  "type": "blockquote",
  "content": [
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "这是引用内容" }
      ]
    }
  ]
}
```

#### 标题 (heading)

```json
{
  "type": "heading",
  "attrs": { "level": 1 },
  "content": [
    { "type": "text", "text": "一级标题" }
  ]
}
```

`level` 取值 1-6，对应 h1-h6。

#### 有序列表 (orderedList) / 无序列表 (bulletList)

```json
{
  "type": "bulletList",
  "content": [
    {
      "type": "listItem",
      "content": [
        {
          "type": "paragraph",
          "content": [
            { "type": "text", "text": "列表项 1" }
          ]
        }
      ]
    }
  ]
}
```

有序列表使用 `"type": "orderedList"`，结构相同。

### 完整 NoteAtom 示例

一篇包含标题、文字、图片和引用的笔记：

```json
{
  "type": "doc",
  "content": [
    {
      "type": "heading",
      "attrs": { "level": 1 },
      "content": [
        { "type": "text", "text": "我的旅行日记" }
      ]
    },
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "今天去了" },
        {
          "type": "text",
          "text": "西湖",
          "marks": [{ "type": "bold" }]
        },
        { "type": "text", "text": "，风景很美！" }
      ]
    },
    {
      "type": "noteImage",
      "attrs": {
        "uuid": "abc123-file-id",
        "width": 1920,
        "height": 1080,
        "ratio": 1.78
      }
    },
    {
      "type": "blockquote",
      "content": [
        {
          "type": "paragraph",
          "content": [
            { "type": "text", "text": "欲把西湖比西子，淡妆浓抹总相宜。" }
          ]
        }
      ]
    }
  ]
}
```

---

## 5. 图片上传

### 5.1 本地上传（两步流程）

适用于上传本地文件。支持格式：gif, jpeg, jpg, png, webp。大小限制：< 50MB。

#### 步骤 1：获取上传授权信息

**POST** `/api/open/api/v1/upload/prepare`

| 属性 | 说明 |
|------|------|
| 限频 | 1次/秒 |
| 配额 | 200次/天 |

请求体：

```json
{
  "fileType": 1,
  "fileName": "photo.jpg"
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `fileType` | integer | ✅ | `1`=图片, `2`=音频, `3`=PDF |
| `fileName` | string | ❌ | 文件名（含扩展名） |

成功响应：

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "form": {
      "endpoint": "https://xxx.oss-cn-xxx.aliyuncs.com",
      "key": "uploads/xxx/photo.jpg",
      "policy": "eyJ...",
      "callback": "eyJ...",
      "success_action_status": "200",
      "x-oss-signature-version": "OSS4-HMAC-SHA256",
      "x-oss-credential": "xxx",
      "x-oss-date": "20250101T000000Z",
      "x-oss-signature": "xxx",
      "x-oss-meta-mo-uid": "xxx",
      "x:file_name": "photo.jpg",
      "x:file_id": "file_abc123",
      "x:file_uid": "xxx"
    }
  }
}
```

#### 步骤 2：文件投递到 OSS

**POST** `{endpoint}/`（endpoint 来自步骤 1 的返回）

- **Content-Type**: `multipart/form-data`
- 将步骤 1 返回的 `form` 中所有字段作为表单字段发送
- 最后追加 `file` 字段，值为实际文件二进制内容

表单字段顺序：

1. `key`
2. `policy`
3. `callback`
4. `success_action_status`
5. `x-oss-signature-version`
6. `x-oss-credential`
7. `x-oss-date`
8. `x-oss-signature`
9. `x-oss-meta-mo-uid`
10. `x:file_name`
11. `x:file_id`
12. `x:file_uid`
13. `file`（文件二进制数据）

成功后，图片的 `fileId` 即为步骤 1 返回的 `form["x:file_id"]`，用于 NoteAtom 中 `noteImage.attrs.uuid`。

### 5.2 远程上传（一步完成）

适用于上传网络图片 URL。大小限制：< 30MB。

**POST** `/api/open/api/v1/upload/url`

| 属性 | 说明 |
|------|------|
| 限频 | 1次/秒 |
| 配额 | 200次/天 |

请求体：

```json
{
  "fileType": 1,
  "url": "https://example.com/photo.jpg",
  "fileName": "photo.jpg"
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `fileType` | integer | ✅ | `1`=图片, `2`=音频, `3`=PDF |
| `url` | string | ✅ | 远程文件 URL |
| `fileName` | string | ❌ | 文件名 |

成功响应：

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "file": {
      "uid": "xxx",
      "fileId": "file_abc123",
      "name": "photo.jpg",
      "path": "https://cdn.mowen.cn/xxx/photo.jpg",
      "type": 1,
      "format": "jpg"
    }
  }
}
```

返回的 `data.file.fileId` 即为 NoteAtom 中 `noteImage.attrs.uuid` 的值。

> ⚠️ 远程上传受目标服务器速度、防盗链策略等影响，不保证一定成功。建议优先使用远程上传，失败时提示用户下载图片后使用本地上传。

---

## 6. 错误码

| HTTP 状态码 | 说明 | 建议 |
|-------------|------|------|
| 200 + `code: 0` | 成功 | - |
| 200 + `code: != 0` | 业务错误 | 查看 `msg` 字段获取详情 |
| 401 | 未认证 | 检查 API-KEY 是否正确 |
| 403 | 无权限 | 检查 API-KEY 权限范围 |
| 429 | 频率限制 | 等待后重试，确保间隔 ≥ 1秒 |
| 500 | 服务器错误 | 稍后重试 |

---

## 7. 限频与配额汇总

| API | 限频 | 日配额 |
|-----|------|--------|
| 笔记创建 | 1次/秒 | 100次/天 |
| 笔记编辑 | 1次/秒 | 1000次/天 |
| 笔记设置 | 1次/秒 | 100次/天 |
| 获取上传授权 | 1次/秒 | 200次/天 |
| 远程上传 | 1次/秒 | 200次/天 |
