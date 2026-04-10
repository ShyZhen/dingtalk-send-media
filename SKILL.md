---
name: dingtalk-send-media
description: 发送钉钉媒体文件（图片/语音/视频/文件）给用户。自动从 OPENCLAW_AGENT_ID 检测账号，跨平台支持。Design by Ash。
emoji: "📤"
requires:
  bins: []  # 无需外部依赖，仅使用 Python 标准库
config:
  read:
    - "~/.openclaw/openclaw.json (channels.dingtalk.accounts 或 channels.dingtalk-connector.accounts, bindings)"
env:
  read:
    - "DINGTALK_CLIENTID (可选，直接指定 clientId，**优先级最高**，从 ~/.openclaw/.env 加载)"
    - "DINGTALK_CLIENTSECRET (可选，直接指定 clientSecret，**优先级最高**，从 ~/.openclaw/.env 加载)"
    - "OPENCLAW_AGENT_ID (可选，用于自动检测账号)"
    - "OPENCLAW_ACCOUNT_ID (可选，直接指定账号)"
    - "OPENCLAW_CONFIG (可选，自定义配置文件路径)"
    - "DINGTALK_ROBOTCODE (可选，默认同 clientId)"
    - "DINGTALK_CORPID (可选，企业 ID)"
    - "DINGTALK_AGENTID (可选，应用 ID)"
---

# DingTalk Send Media - 钉钉媒体文件发送技能

## 触发场景

当用户需要：
- 发送图片/文件/语音/视频/附件给用户
- 自动从当前会话检测钉钉账号
- 跨平台发送媒体文件（Windows/Linux/macOS）

## 特性

✅ **自动账号检测** - 从 `OPENCLAW_AGENT_ID` 自动推导钉钉账号  
✅ **零依赖** - 仅使用 Python 标准库，无需 `curl`/`jq`  
✅ **跨平台** - Windows/Linux/macOS 通用  
✅ **多媒体支持** - 自动检测 `image` / `voice` / `video` / `file`  
✅ **正确的文件名** - 自动传递 `fileName` 参数，避免显示 `#fileName#`

## 使用方法

### 方式 1：直接调用脚本（推荐）

```bash
# 进入技能目录
cd ~/.openclaw/skills/dingtalk-send-media

# 发送文件（自动检测账号）
python scripts/send_media.py <文件路径> <目标 ID>

# 发送文件（指定账号）
python scripts/send_media.py <文件路径> <目标 ID> [账号 ID] [媒体类型]
```

**参数说明**：
- `文件路径`: 本地文件的绝对路径
- `目标 ID`: 钉钉用户 ID（如 `300523656829570034`）或群 ID（以 `cid` 开头）
- `账号 ID`: OpenClaw 配置中的钉钉账号 ID（可选，默认从 `OPENCLAW_AGENT_ID` 自动检测）
- `媒体类型`: `image` / `voice` / `video` / `file`（可选，默认自动检测）

### 方式 2：在对话中使用

告诉模型你要发送的文件，模型会自动调用脚本。

**示例**：
```bash
# 自动检测账号，发送文件
python scripts/send_media.py "C:/path/to/file.pdf" "300523656829570034"

# 指定账号发送
python scripts/send_media.py "C:/path/to/image.png" "300523656829570034" prd_bot

# 指定媒体类型
python scripts/send_media.py "C:/path/photo.jpg" "300523656829570034" prd_bot image

# 发送到群聊
python scripts/send_media.py "C:/path/to/meeting.mp4" "cidxxxxxxxx" dev_bot video
```

## 典型场景

### 场景 1：发送报告文件

**用户**: "把昨天的销售报告发给我"

**模型**:
```bash
python scripts/send_media.py "C:/path/to/reports/sales_report_2026-04-06.pdf" "300523656829570034"
```

**输出**:
```json
{
  "ok": true,
  "mediaId": "@lAjPD0...",
  "result": {"processQueryKey": "xxx="},
  "file": "sales_report_2026-04-06.pdf",
  "target": "300523656829570034",
  "type": "file",
  "account": "prd_bot"
}
```

### 场景 2：发送截图

**用户**: "把这个错误的截图发给我看看"

**模型**:
```bash
python scripts/send_media.py "C:/path/to/screenshots/error_2026-04-07.png" "300523656829570034"
```

### 场景 3：发送到群聊

**用户**: "把会议纪要发到项目群"

**模型**:
```bash
python scripts/send_media.py "C:/path/to/meeting_notes.docx" "cidxxxxxxxx" prd_bot file
```

### 场景 4：发送语音/视频

**用户**: "把这个录音发给客户"

**模型**:
```bash
python scripts/send_media.py "C:/path/to/recording.mp3" "300523656829570034" prd_bot voice
```

## 自动账号检测原理

### 账号检测优先级

脚本通过以下**优先级**自动检测钉钉账号：

1. **显式参数** - 命令行直接指定账号 ID（最高优先级）
2. **`OPENCLAW_ACCOUNT_ID`** - 环境变量直接指定账号
3. **`OPENCLAW_AGENT_ID` + bindings** - 从当前 Agent 推导账号
4. **Agent ID 后缀** - 从 `dingtalk-xxx` 提取 `xxx`
5. **默认值** - `prd_bot`（最低优先级）

### 环境变量说明

#### OpenClaw 相关

| 变量 | 说明 | 示例 |
|------|------|------|
| `OPENCLAW_AGENT_ID` | 当前 Agent ID，由 OpenClaw 运行时设置 | `main`, `developer`, `dingtalk-office` |
| `OPENCLAW_ACCOUNT_ID` | **直接指定**钉钉账号 ID（优先级最高） | `prd_bot`, `dev_bot` |
| `OPENCLAW_CONFIG` | 自定义配置文件路径 | `/etc/openclaw/openclaw.json` |

#### 钉钉凭证（**优先级最高**）

**配置方式：** 在 `~/.openclaw/.env` 文件中添加（OpenClaw 会自动加载）

```bash
# ~/.openclaw/.env
DINGTALK_CLIENTID=dingxxxxxx
DINGTALK_CLIENTSECRET=your_secret
DINGTALK_ROBOTCODE=dingxxxxxx  # 可选，默认同 clientId
```

| 变量 | 说明 | 必需 | 默认值 |
|------|------|------|--------|
| `DINGTALK_CLIENTID` | 钉钉应用 ClientId | ✅ | - |
| `DINGTALK_CLIENTSECRET` | 钉钉应用 ClientSecret | ✅ | - |
| `DINGTALK_ROBOTCODE` | 机器人代码 | ❌ | 同 clientId |
| `DINGTALK_CORPID` | 企业 ID | ❌ | - |
| `DINGTALK_AGENTID` | 应用 ID | ❌ | - |

**优先级说明：**
- ✅ **环境变量优先级最高** — 设置后会覆盖配置文件中的值
- ✅ **OpenClaw 自动加载** `~/.openclaw/.env`，无需手动读取
- ✅ 适合临时切换凭证（如测试/生产环境）
- ✅ 避免在配置文件中暴露敏感信息

**使用场景：**
- 纯环境变量部署（如 Docker、CI/CD）
- 不想在配置文件中暴露敏感信息
- 多环境切换（开发/测试/生产）
- 临时使用不同账号测试

### 调试模式

使用 `--debug` 或 `-d` 参数查看账号检测过程：

```bash
python scripts/send_media.py "file.pdf" "user_id" --debug
```

输出示例：
```
[DEBUG] OPENCLAW_AGENT_ID=main
[DEBUG] OPENCLAW_ACCOUNT_ID=未设置
[DEBUG] 账号检测来源：binding:main→prd_bot
```

**示例配置**：

```powershell
# 方式 1：环境变量（优先级最高 ⭐）
$env:DINGTALK_CLIENTID="dingxxxxxx"
$env:DINGTALK_CLIENTSECRET="your_secret"
$env:DINGTALK_ROBOTCODE="dingxxxxxx"  # 可选，默认同 clientId

python scripts/send_media.py "file.pdf" "300523656829570034"
```

```json
// 方式 2：openclaw.json 配置
{
  "bindings": [
    {
      "agentId": "main",
      "match": {
        "channel": "dingtalk-connector",
        "accountId": "prd_bot"
      }
    }
  ],
  "channels": {
    "dingtalk-connector": {
      "accounts": {
        "prd_bot": {
          "clientId": "dingxxxxxx",
          "clientSecret": "xxx",
          "robotCode": "dingxxxxxx"
        }
      }
    }
  }
}
```

```json
// 方式 3：配置文件 + 环境变量占位符
{
  "channels": {
    "dingtalk": {
      "accounts": {
        "prd_bot": {
          "clientId": "${DINGTALK_CLIENTID}",
          "clientSecret": "${DINGTALK_CLIENTSECRET}"
        }
      }
    }
  }
}
```

**优先级对比**：
| 方式 | 优先级 | 适用场景 |
|------|--------|----------|
| 环境变量 `DINGTALK_*` | ⭐⭐⭐ 最高 | 临时切换、CI/CD、Docker |
| 配置文件 `${VAR}` 占位符 | ⭐⭐ 中 | 统一配置管理 |
| 配置文件显式值 | ⭐ 低 | 固定配置 |

## 注意事项

1. **路径格式**：
    - Windows: 使用正斜杠 `/` 或双反斜杠 `\\`，路径用引号包裹
    - Linux/macOS: 使用正斜杠 `/`

2. **文件存在性**：确保文件在发送时存在于指定路径

3. **文件大小**：钉钉对文件大小有限制
    - 文件：最大 20MB
    - 图片：最大 10MB
    - 语音：最大 2MB
    - 视频：最大 20MB

4. **文件类型**：支持常见格式
    - 文件：PDF, DOCX, XLSX, PPTX, TXT, ZIP 等
    - 图片：JPG, PNG, GIF, BMP, WEBP
    - 语音：MP3, WAV, AMR
    - 视频：MP4, AVI, MOV

5. **权限要求**：
    - 钉钉应用需要开通"机器人消息发送相关权限"
    - 需要开通"媒体文件上传相关权限"
    - 配置中的 `robotCode` 必须正确设置

6. **目标 ID 格式**：
    - 用户 ID: 钉钉用户 ID（如 `300523656829570034`）
    - 群 ID: 以 `cid` 开头（如 `cidxxxxxxxx`）

## 跨平台说明

| 平台 | 路径示例 | Python 命令 |
|------|---------|-------------|
| Windows | `C:/path/to/file.pdf` 或 `C:\\path\\to\\file.pdf` | `python` |
| Linux | `/path/to/file.pdf` | `python3` |
| macOS | `/path/to/file.pdf` | `python3` |

## 测试状态

✅ **脚本已测试通过** (2026-04-08)
- 文件上传：成功
- 消息发送：成功
- 文件名显示：正确（非 `#fileName#`）
- 测试文件：`test_markdown.md`, `test88.json`
- 测试目标：用户 ID `300523656829570034`

## 相关文件

| 文件 | 说明 |
|------|------|
| `SKILL.md` | 技能说明文档（本文件） |
| `scripts/send_media.py` | 钉钉媒体发送主脚本 |

## 依赖说明

- **Python**: 需要 Python 3.6+
- **第三方库**: 无（仅使用 Python 标准库）
- **环境变量**: 钉钉凭证可能需要在环境变量中配置

## 与 dingtalk-file-send 对比

| 特性 | dingtalk-send-media | dingtalk-file-send |
|------|---------------------|-------------------|
| 实现语言 | Python | Bash |
| 外部依赖 | 无 | `curl`, `jq` |
| Windows 兼容 | ✅ 原生支持 | ❌ 需要 Git Bash/WSL |
| 自动账号检测 | ✅ 支持 | ✅ 支持 |
| 多媒体类型 | image/voice/video/file | 仅 file |
| OpenClaw 元数据 | ✅ 支持 | ✅ 支持 |

## 故障排除

### 问题 1: 找不到配置文件
```
错误：未找到 OpenClaw 配置文件 openclaw.json
```
**解决**: 确认 `~/.openclaw/openclaw.json` 存在，或设置 `OPENCLAW_CONFIG` 环境变量

### 问题 2: 获取 access token 失败
```
错误：获取 access token 失败：...
```
**解决**: 检查 `clientId` 和 `clientSecret` 是否正确，确认钉钉应用已发布

### 问题 3: 上传文件失败
```
错误：上传媒体文件失败：...
```
**解决**:
- 确认文件路径正确且文件存在
- 确认文件大小不超过限制
- 确认钉钉应用有媒体上传权限

### 问题 4: 发送消息失败
```
错误：发送消息失败：...
```
**解决**:
- 确认目标用户 ID 或群 ID 正确
- 确认机器人已添加到群聊（群聊场景）
- 确认 `robotCode` 配置正确

### 问题 5: 文件名显示为 #fileName#
**已修复** - 脚本现在自动传递 `fileName` 参数

### 问题 6: 账号检测错误
```
错误：未找到钉钉账号配置：xxx
```
**解决**:
- 检查 `openclaw.json` 中的 `channels.dingtalk.accounts` 或 `channels.dingtalk-connector.accounts` 配置
- 检查 `bindings` 中的 `agentId → accountId` 映射
- 或设置环境变量 `DINGTALK_CLIENTID` + `DINGTALK_CLIENTSECRET`
- 或显式指定账号 ID 参数

### 问题 7: 纯环境变量模式
```
错误：未找到钉钉账号配置，请在 openclaw.json 中配置，或设置环境变量
```
**解决**:
- 设置环境变量（优先级最高）：
  ```powershell
  $env:DINGTALK_CLIENTID="dingxxxxxx"
  $env:DINGTALK_CLIENTSECRET="your_secret"
  ```
- 脚本会自动使用环境变量中的凭证，覆盖配置文件

### 问题 8: 环境变量覆盖配置
**说明**: 当同时设置了环境变量和配置文件时，**环境变量优先级更高**。

**示例**:
```json
// openclaw.json 配置了 test_bot
{
  "channels": {
    "dingtalk": {
      "accounts": {
        "test_bot": {
          "clientId": "ding_test",
          "clientSecret": "test_secret"
        }
      }
    }
  }
}
```

```powershell
# 但环境变量设置了 prod_bot 的凭证
$env:DINGTALK_CLIENTID="ding_prod"
$env:DINGTALK_CLIENTSECRET="prod_secret"

# 实际会使用环境变量的 prod_bot 凭证
python scripts/send_media.py "file.pdf" "user_id"
```

---

_最后更新：2026-04-10_
