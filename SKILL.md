---
name: sensenova-image-gen
description: Generate and edit images via SenseNova U1.5 Lite model (商汤日日新) using curl to call the text-to-image and image-editing APIs. Trigger when the user mentions 商汤, SenseNova, U1.5, 文生图, 图生图, text-to-image generation, image editing, or wants to create/edit images with this specific model.
---

# SenseNova U1.5 Lite 图像生成与编辑

通过商汤 SenseNova U1.5 Lite 模型 API，实现文生图（文本生成图片）与图生图（图片编辑/改写）。

---

## 快速开始

### 1. 配置 Token

首次使用需要 API Token：

1. **检查已有 Token**：在 MEMORY.md 的「工具设置」部分查找 `sensenova-image-gen token`，若已存在则直接使用。
2. **获取 Token**：若未配置，请用户提供其商汤 API Token（格式：`Authorization: Bearer <token>`）。
3. **保存 Token**：将 Token 写入 MEMORY.md 的「工具设置」部分：
   ```
   ### sensenova-image-gen
   - token: <the-token-value>
   ```
4. 确认已保存，后续调用无需重复提供。

### 2. 文生图

根据用户描述构建请求并执行：

```bash
curl -s --max-time 120 --retry 3 --retry-delay 5 https://token.sensenova.cn/v1/images/generations \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sensenova-u1.5-lite",
    "prompt": "你的图片描述",
    "size": "2048x2048",
    "n": 1,
    "watermark": false,
    "response_format": "url"
  }'
```

### 3. 图生图 / 图片编辑

传入参考图片 + 编辑指令：

```bash
curl -s --max-time 120 --retry 3 --retry-delay 5 https://token.sensenova.cn/v1/images/edits \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sensenova-u1.5-lite",
    "images": [
      { "image_url": "https://example.com/input.png" }
    ],
    "prompt": "把背景换成海边日落",
    "n": 1,
    "response_format": "url"
  }'
```

### 4. 下载并展示

若 `response_format` 为 `url`，将图片下载到本地：

```bash
curl -s -o output.png "<image_url>"
```

若 `response_format` 为 `b64_json`（默认），从响应中提取 Base64 并解码保存为图片文件。

将图片展示给用户或发送给用户。

---

## API 参考

### 端点

```
POST https://token.sensenova.cn/v1/images/generations   # 文生图
POST https://token.sensenova.cn/v1/images/edits         # 图生图 / 图片编辑
```

### 请求头

```
Authorization: Bearer {token}
Content-Type: application/json
```

### 文生图请求参数（/v1/images/generations）

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `model` | string | 是 | — | 固定值 `sensenova-u1.5-lite` |
| `prompt` | string | 是 | — | 图像描述文本 |
| `size` | string | 否 | `auto` | 图像尺寸，见下方尺寸表 |
| `n` | integer | 否 | 1 | 生成图片数量，**仅支持 1** |
| `watermark` | boolean | 否 | `true` | 是否添加日日新官方 Logo 水印；`false` 生成无水印纯图（当前免费公测） |
| `output_format` | string | 否 | `png` | 图片文件格式：`png` / `jpeg` / `webp` |
| `response_format` | string | 否 | `b64_json` | 结果承载方式：`b64_json`（返回 Base64）/ `url`（返回 24 小时有效临时链接） |
| `prompt_extend` | boolean | 否 | `true` | 提示词自动润色优化开关 |

### 图生图请求参数（/v1/images/edits）

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `model` | string | 是 | — | 固定值 `sensenova-u1.5-lite` |
| `images` | array | 是 | — | 图片对象数组，每项含 `image_url`；第一张为主编辑图 |
| `images[].image_url` | string | 是 | — | 公网 HTTP/HTTPS URL 或 Base64 Data-URL（`data:image/png;base64,xxx`）；不支持纯无前缀 Base64 |
| `prompt` | string | 是 | — | 编辑指令，描述期望的最终画面 |
| `n` | integer | 否 | 1 | 仅支持 1 |
| `size` | string | 否 | `auto` | 默认自动适配主图尺寸 |
| `response_format` | string | 否 | `b64_json` | `b64_json` / `url` |
| `watermark` | boolean | 否 | `true` | 是否添加水印 |
| `prompt_extend` | boolean | 否 | `true` | 提示词自动润色开关 |

### 尺寸与比例对照表

`sensenova-u1.5-lite` 支持 2K / 4K 分辨率；WIDTH 和 HEIGHT 需为 32 的倍数，最小值 512，最大值 4096，最大比例 3:1 或 1:3。

| 用途建议 | 尺寸 | 比例 | 分辨率 |
|----------|------|------|--------|
| 人像 / 竖版海报 | `1664x2496` | 2:3 | 2K |
| 风景 / 横版展示 | `2496x1664` | 3:2 | 2K |
| 手机壁纸 / 竖版插画 | `1536x2720` | 9:16 | 2K |
| 桌面壁纸 / 宽屏展示 | `2720x1536` | 16:9 | 2K |
| **头像 / 图标 / 通用** | **`2048x2048`** | **1:1** | **2K** |
| 超高清 / 大图输出 | `4096x4096` | 1:1 | 4K |

> 用户未指定尺寸时默认 `auto`（模型自动选择），或根据用途从表中推荐。

---

## 响应格式

### 成功响应（response_format=url）

```json
{
  "created": 1700000000,
  "data": [
    { "url": "https://..." }
  ]
}
```

提取图片 URL：`response.data[0].url`

### 成功响应（response_format=b64_json，默认）

```json
{
  "created": 1700000000,
  "data": [
    { "b64_json": "iVBORw0KGgo..." }
  ]
}
```

提取 Base64：`response.data[0].b64_json`，解码后保存为图片文件。

---

## 错误处理

### 重试策略

- 网络超时或连接失败时，**自动重试最多 3 次**，每次间隔 5 秒。
- curl 命令中已包含 `--retry 3 --retry-delay 5 --max-time 120` 参数（U1.5 Lite 生成时间较长，超时放宽到 120 秒）。

### 常见错误码

| HTTP 状态码 | 错误类型 | 含义 | 处理方式 |
|--------|----------|------|----------|
| `400` | `invalid_request_error` | 请求参数不合法（缺失、超范围、格式错误） | 检查 prompt、size（32 的倍数、512-4096）、images 格式 |
| `400` | `failed_precondition_error` | 前置条件不满足（编码失败、安全检查未通过） | 检查图片 URL/Base64 是否有效、内容是否合规 |
| `403` | `permission_denied_error` | Token 无效或权限不足 | 提醒用户检查 Token |
| `404` | `not_found_error` | 模型 ID 不存在或已下线 | 确认 model 为 `sensenova-u1.5-lite` |
| `408` | `canceled_error` | 客户端取消请求 | 重新发起请求 |
| `429` | `quota_exceeded_error` | 速率/额度超限 | 等待后重试（指数退避），或提醒用户稍后再试 |
| `500` | `internal_server_error` | 服务器内部错误 | 等待后重试；持续出现则提醒用户服务可能不可用 |

### 错误处理流程

1. 执行 curl 命令后，首先检查 HTTP 状态码。
2. 若状态码非 `200`，解析响应体中的错误信息，按上表处理。
3. 对于可重试错误（429、500、网络超时），自动等待后重试，最多 3 次。
4. 重试仍失败时，向用户展示**友好的中文错误提示**，包含错误原因、建议解决方式、原始错误信息。

---

## 使用场景示例

### 示例 1：生成头像

> 用户说：「帮我生成一个赛博朋克风格的头像」

- 尺寸选择：`2048x2048`（1:1，适合头像）
- prompt：`赛博朋克风格的头像，霓虹灯光，未来感，数字艺术`

### 示例 2：生成桌面壁纸

> 用户说：「画一张山间日出的风景壁纸」

- 尺寸选择：`2720x1536`（16:9，适合桌面壁纸）
- prompt：`山间日出的壮丽风景，金色阳光穿透云层，远山层叠，高清摄影风格`

### 示例 3：生成手机壁纸

> 用户说：「做一个可爱的猫咪手机壁纸」

- 尺寸选择：`1536x2720`（9:16，适合手机竖屏）
- prompt：`可爱的橘猫趴在窗台上，阳光洒落，温馨治愈风格，插画`

### 示例 4：图片编辑（图生图）

> 用户说：「把这张照片的背景换成海边日落」

- 使用 `/v1/images/edits` 接口，传入原图 URL + 编辑指令
- prompt：`把背景换成海边日落，暖色调，保持人物主体不变`

### 示例 5：生成 4K 高清大图

> 用户说：「生成一张 4K 高清的星空图」

- 尺寸选择：`4096x4096`（1:1，4K）
- prompt：`壮丽的银河星空，繁星点点，深蓝色调，超高清细节`

---

## 常见问题（FAQ）

**Q：Token 从哪里获取？**
A：前往商汤 SenseNova 官网（https://platform.sensenova.cn）注册账号，在控制台创建 API Key 即可获取。

**Q：生成的图片链接有效期多久？**
A：`response_format=url` 返回的临时链接有效期为 **24 小时**，建议生成后**立即下载保存**到本地。

**Q：一次最多生成几张图片？**
A：`sensenova-u1.5-lite` 的 `n` 参数**仅支持 1**，单次只能生成一张图片。

**Q：如何生成无水印图片？**
A：设置 `watermark: false`。该功能当前免费公测，后续将转为付费特性，建议显式传入该参数。

**Q：支持哪些格式的图片输出？**
A：`output_format` 支持 `png`、`jpeg`、`webp`。PNG 适合透明背景和无损画面，JPEG 适合照片类图片（不支持透明背景），WEBP 兼顾文件大小和透明背景。

**Q：图生图接口支持什么形式的图片输入？**
A：支持公网 HTTP/HTTPS URL 和 Base64 Data-URL（`data:image/png;base64,xxx`），**不支持**纯无前缀的 Base64 字符串。

**Q：prompt 有什么技巧？**
A：描述越具体越好。推荐结构：`主体 + 场景/背景 + 风格 + 细节修饰`。`prompt_extend` 默认开启提示词自动润色，可提升生成效果。

---

## 注意事项

- Token 保存后自动复用，用户只需提供一次。
- 尊重用户的语言偏好，用用户使用的语言进行 `prompt` 描述。
- 用户未指定尺寸时，根据用途智能推荐（见尺寸对照表），或使用默认 `auto`。
- 图片 URL 有时效性（24 小时），生成后应立即下载保存。
- 该模型支持文生图与图生图两种能力：纯文本描述用 `/v1/images/generations`，带参考图编辑用 `/v1/images/edits`。
- 遇到错误时，优先自动重试，重试失败后给出友好的中文提示和解决建议。
