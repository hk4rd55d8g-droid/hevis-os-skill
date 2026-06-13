---
name: "hevis-os-official-skill"
description: "Hevis-OS 认知操作系统官方 Skill，提供人/事/看/备/存/搜/记/摄/连/标 10 大能力域的统一接入、连接器配置与激活码管理，支持 Qoder/WorkBuddy/Coze/Dify/ChatGPT 多平台一键集成"
version: "1.2.0"
license: "MIT"
metadata:
  author: "Hevis-OS Team"
  spc_ref: "SPC §6 — 连能力域"
  skill_id: "hevis-os-skill"
  min_connector_version: "1.0.0"
  compatible_versions: ["1.0.0", "1.1.0", "1.2.0"]
---

# Hevis-OS 官方 Skill 完整说明

> 本文档遵循「好 Skill 的 8 层架构」标准编写。
> 每一层依次递进，从身份识别到可执行的完整示例，覆盖此 Skill 的全部职责。

---

## 1️⃣ 身份 (Identity)

| 字段 | 值 |
|---|---|
| Skill 名称 | `hevis-os-official-skill` |
| Skill ID | `hevis-os-skill` |
| 版本 | `1.2.0` |
| 协议 | MIT |
| SPC 引用 | SPC §6 — 连 (Connect) 能力域 |
| 能力域总数 | 10 个（人/事/看/备/存/搜/记/摄/连/标） |
| 支持平台 | 4 个（Qoder/WorkBuddy、Coze、Dify、ChatGPT） |
| 连接器最低版本 | `1.0.0` |
| 兼容版本 | `1.0.0`, `1.1.0`, `1.2.0` |

---

## 2️⃣ 定义 (Definition)

Hevis-OS 官方 Skill 是 Hevis-OS 认知操作系统对外分发的标准化智能体能力包。它定义了外部 Agent（无论运行在何种平台上）如何接入 Hevis-OS 组织记忆系统、获取哪些能力域的访问权限、以及如何维持连接活性。

**核心职责分离：**
- **Agent（外部智能体）**：决策判断——什么时候调用哪个能力域
- **Skill（能力包）**：标准化执行——定义接入流程、权限范围、心跳维持、平台配置

**两大核心子系统：**

### 2.1 能力域激活系统（10 域）
外部 Agent 安装此 Skill 后，可根据授权获取以下能力域的访问权限：

| 域 | ID | 中文说明 | 核心 APIs |
|---|---|---|---|
| 👤 人 | person | 人物卡片、关系网络、评分管理 | memory.read, memory.write |
| 📋 事 | task | 项目、商机、任务、供应商管理 | memory.read, memory.write, workflow.execute |
| 📊 看 | brief | 每日简报、使用分析、运营数据 | memory.read, context.inject |
| 🎯 备 | prepare | 会前准备、材料整理、会议支持 | workflow.bootstrap, workflow.plan, workflow.execute |
| 📁 存 | store | 文档结构化、知识库管理、归档 | memory.write, memory.read |
| 🔍 搜 | search | 全网知识搜索、知识库检索 | memory.read |
| 🎙️ 记 | record | 录音、会议记录、AI 分析、纪要生成 | memory.write, workflow.execute |
| 📥 摄 | ingest | 数据摄入：系统分享、相机、扫描、剪贴板、文件 | memory.write |
| 🔗 连 | connect | 第三方平台连接、Skill 码管理、外部 Agent 接入 | memory.read, memory.write |
| 🎯 标 | target | OKR 目标管理、目标网络、关键成果追踪 | memory.read, memory.write |

### 2.2 连接器与激活码管理（SPC §6）
外部 Agent 必须经过 **「生成激活码 → 凭码激活 → 心跳保活」** 三步骤完成接入，确保每一次连接都有明确的组织身份归属。

---

## 3️⃣ 场景 (When to use / When NOT to use)

### ✅ When to use（适用场景）

**场景一：首次安装外部 Agent**
> 用户在 Coze/Dify/ChatGPT/Qoder 平台上创建了一个新的 Bot/AI Agent，希望它能访问 Hevis-OS 组织记忆。

此时需要：
1. 在 Hevis-OS 管理后台生成一个 Skill Code
2. 将此 Skill Code 复制到外部 Agent 平台进行激活
3. 下载对应平台的配置文件，完成连接器设置

**场景二：外部 Agent 能力升级**
> 已连接的外部 Agent 需要扩展能力域（如从只读升级为读写权限）。

此时需要：
1. 在服务端更新 Agent 的授权权限
2. Agent 重新拉取能力声明
3. Agent 获取新的工具/API 调用权限

**场景三：连接健康检查与保活**
> 需要确认已接入的外部 Agent 是否仍在正常运行。

此时 Agent 应定时发送心跳信号，服务端检测心跳超时（15 分钟）的连接，标记为失活（stale）。

**场景四：接入管理**
> 管理员需要查看哪些外部 Agent 已接入、哪些 Code 已激活、哪些绑定已离线。

### ❌ When NOT to use（不适用的场景）

- 用户只需要在 Hevis-OS 原生 App（Android 客户端）内操作，不涉及外部 Agent 接入
- 用户需要的是组织记忆的直接读写操作（应使用具体的「人/事/存/搜」等能力域）
- 用户需要的是系统管理后台配置（应使用管理员控制台）
- 用户需要在没有网络的环境中运行（此 Skill 依赖服务端 API）

---

## 4️⃣ 输入 (Input)

### 4.1 激活码生成（服务端）

```
Method:  POST /api/v1/member/codes
Auth:    Member 身份（登录态）
Body:    无
```

### 4.2 激活码激活（第三方 Agent）

```
Method:  POST /api/v1/skill.activate
Auth:    无（凭 code 认证）
Body:    {
           "code": "mcode_<24-char-hex>",
           "platform": "coze | dify | chatgpt | qoder | qoder-workbuddy",
           "platform_user_id": "<第三方平台用户唯一标识>",
           "skill_key": "<Agent 在第三方平台的标识>"
         }
```

### 4.3 心跳保活

```
Method:  POST /api/v1/skill.heartbeat
Auth:    无（凭 binding 信息认证）
Body:    {
           "skill_key": "<Agent 标识>",
           "platform": "coze | dify | chatgpt | qoder | qoder-workbuddy",
           "platform_user_id": "<第三方平台用户唯一标识>"
         }
```

### 4.4 连接器配置文件下载（第三方 Agent）

```
Method:  GET /api/v1/skill/config/:platform/:code
Auth:    无（凭 code 认证）
Params:  platform  → qoder-workbuddy | coze | dify | chatgpt
         code      → mcode_<24-char-hex>
```

### 4.5 失活绑定查询（管理员）

```
Method:  GET /api/v1/skill.stale-bindings
Auth:    Member 身份（登录态）
Body:    无
```

---

## 5️⃣ 输出 (Output)

### 5.1 激活码生成响应

```json
{
  "id": "sc_20260530120000",
  "code": "mcode_a1b2c3d4e5f6a7b8c9d0e1f2",
  "status": "pending",
  "member_id": "member_uuid_xxx",
  "created_at": "2026-05-30T12:00:00Z",
  "expires_at": "2026-05-30T12:15:00Z"
}
```

### 5.2 激活成功响应

```json
{
  "id": "sb_20260530120100",
  "skill_key": "my-coze-bot-v1",
  "member_id": "member_uuid_xxx",
  "platform": "coze",
  "platform_user_id": "coze_user_xxx",
  "status": "active",
  "created_at": "2026-05-30T12:01:00Z"
}
```

### 5.3 心跳响应

```json
{
  "id": "sb_20260530120100",
  "skill_key": "my-coze-bot-v1",
  "status": "active",
  "last_heartbeat": "2026-05-30T12:05:00Z",
  "created_at": "2026-05-30T12:01:00Z"
}
```

### 5.4 平台连接器配置文件

根据平台不同，输出以下格式之一的配置文件：
- **Qoder/WorkBuddy**: MCP JSON（Structured Output Server 配置）
- **Coze**: OpenAPI YAML（插件配置）
- **Dify**: Plugin YAML（工具配置）
- **ChatGPT**: Actions JSON（GPTs Action 配置）

所有配置均包含：
- `X-Member-ID` 头认证机制（值 = 激活码所属成员 ID）
- Hevis-OS 服务端 API 端点地址
- 可用工具列表

### 5.5 安装页面输出

`GET /skill/install/:code` 返回 HTML 安装引导页，包含：
- 4 个平台的配置文件下载链接
- 分平台接入步骤说明
- 激活码状态与所属成员信息

---

## 6️⃣ 工作流 (Instructions)

### 工作流 1：首次接入 —— 生成 → 激活 → 配置 → 使用

```
步骤 1 — 生成激活码
  ├── 用户在 Hevis-OS 管理后台或 App 中调用「生成 Skill Code」
  │   POST /api/v1/member/codes
  ├── 系统生成 mcode_ 开头的 24 位十六进制码（15 分钟有效）
  └── 用户复制得到的 Code 字符串

步骤 2 — 打开安装引导页
  ├── 用户访问 https://hevis.zwkjxm.com/skill/install/<Code>
  ├── 页面显示 4 个平台的配置下载入口
  └── 用户选择目标平台（Qoder/Coze/Dify/ChatGPT）

步骤 3 — 下载平台配置文件
  ├── 点击对应平台的下载按钮
  │   GET /api/v1/skill/config/<platform>/<Code>
  └── 服务端验证 Code 有效后，返回平台特定的配置文件

步骤 4 — 在目标平台配置 Agent
  根据平台选择对应的操作：
  ├── Qoder/WorkBuddy → 将 MCP JSON 导入为 MCP Server 配置
  ├── Coze            → 导入 OpenAPI YAML 作为插件
  ├── Dify            → 导入 Plugin YAML 作为工具
  └── ChatGPT         → 导入 Actions JSON 作为 GPTs Action

步骤 5 — 激活连接
  ├── Agent 在目标平台启动后，调用激活 API：
  │   POST /api/v1/skill.activate
  │   Body: { "code": "<Code>", "platform": "...",
  │           "platform_user_id": "...", "skill_key": "..." }
  ├── 状态变为 bound，创建 SkillBinding
  └── Agent 开始通过 MCP/OpenAPI 访问 Hevis-OS 能力域
```

### 工作流 2：连接保活（Agent 端定时执行）

```
每隔 ≤15 分钟执行一次：
  ├── Agent 调用 POST /api/v1/skill.heartbeat
  │   Body: { "skill_key": "...", "platform": "...", "platform_user_id": "..." }
  ├── 服务端更新该 Binding 的 last_heartbeat 时间戳
  └── 若超过 15 分钟未收到心跳，Binding 标记为 stale

管理员可查询失活绑定：
  └── GET /api/v1/skill.stale-bindings
  └── 返回所有超过 15 分钟未发心跳的 active 绑定
```

### 工作流 3：能力域调用（Agent 运行时）

```
Agent 根据用户意图自动选择能力域：
  ├── 用户提问关于某个人的信息 → 使用 person 域 → hevis_memory_read
  ├── 用户要求记录一条信息     → 使用 store/record → hevis_memory_write
  ├── 用户搜索知识库           → 使用 search 域 → hevis_knowledge_query
  ├── 用户要求查看今日简报     → 使用 brief 域 → GET /api/v1/member/brief
  └── 用户安排一个会议         → 使用 prepare 域 → workflow 系列 APIs

认证方式：所有 API 请求携带 X-Member-ID 头，值为激活码对应成员 ID。
```

### 工作流 4：连接生命周期管理

```
管理员在管理后台进行操作：
  ├── 查看已生成的激活码列表 → GET /api/v1/member/codes
  ├── 撤销一个未使用的激活码 → DELETE /api/v1/member/codes/:id
  ├── 撤销一个 Agent 绑定   → DELETE /api/v1/member/bindings/:skill_key
  └── 查看失活绑定净化连接   → GET /api/v1/skill.stale-bindings
```

---

## 7️⃣ 规则 (Rules/Notes)

### 7.1 激活码规则
- **格式**：`mcode_` + 24 位十六进制字符（12 字节随机数）
- **有效期**：自生成起 15 分钟（SPC §6.5.1），超时自动标记 expired
- **状态机**：`pending → bound | expired | revoked`
- **一次性**：一个 code 只能被激活一次
- **撤销不可逆**：revoked 状态不可恢复为 pending

### 7.2 绑定规则
- **一码一平台**：一个激活码成功激活后即绑定到指定的 platform+platform_user_id
- **存活判定**：15 分钟内未收到心跳的 Binding 标记为 stale
- **状态机**：`active → revoked`（撤销不可逆）

### 7.3 连接器版本兼容规则
- **最低版本要求**：`1.0.0`
- **兼容版本列表**：`1.0.0`, `1.1.0`, `1.2.0`
- **升级**：发布新版本 Release 时，通过 Webhook 通知服务器更新 capabilities.yaml 元数据（updated_at, updated_by）
- **不兼容**：如果连接器版本低于 min_version，Agent 应拒绝连接或提示升级

### 7.4 安全规则
- **认证**：MCP/API 请求使用 `X-Member-ID` 头标识调用者身份
- **无密钥传输**：配置文件不传输明文密钥，认证信息为自动注入的 MemberID
- **Code 即密钥**：激活码本身作为一次性临时凭证，不应明文记录在 Agent 配置中
- **Webhook 签名**：GitHub Release Webhook 使用 `X-Hub-Signature-256` HMAC-SHA256 验证来源

### 7.5 平台支持约束
- **Qoder/WorkBuddy**：MCP JSON 格式，支持完整工具列表
- **Coze**：OpenAPI 3.0 YAML 格式，支持 API Key 头认证
- **Dify**：Plugin YAML 格式，自定义工具定义
- **ChatGPT**：GPTs Action JSON 格式，OpenAPI 3.0 子集

### 7.6 限流与配额规则
- 一个成员同时可生成的 pending code 数量上限待配置（默认无限制）
- 心跳频率建议 `5-10 分钟/次`，不应短于 1 分钟
- 能力域 API 调用频率受服务端全局限流控制

---

## 8️⃣ 示例 (Examples)

### 示例 1：完整接入流程（Coze 平台）

**用户操作：**
```
1. 在 Hevis-OS App 点击「生成 Skill Code」
   → 获得 code = mcode_a1b2c3d4e5f6a7b8c9d0e1f2

2. 浏览器打开 https://hevis.zwkjxm.com/skill/install/mcode_a1b2c3d4e5f6a7b8c9d0e1f2
   → 页面显示 4 个平台选项

3. 点击「Coze」下载按钮
   → 获得 hevis-coze-openapi.yaml

4. 在 Coze Bot 配置页面导入 OpenAPI YAML 文件
   → Bot 自动获得 hevis_search、get_brief 等工具

5. Bot 启动后自动调用激活 API：
   POST /api/v1/skill.activate
   {
     "code": "mcode_a1b2c3d4e5f6a7b8c9d0e1f2",
     "platform": "coze",
     "platform_user_id": "coze_user_abc123",
     "skill_key": "my-coze-assistant-v2"
   }
   → Response: { "status": "active", "id": "sb_20260530120100" }

6. 用户现在可以问 Bot："我今天有什么待办任务？"
   → Bot 调用 GET /api/v1/mcp (hevis_memory_read, domain=task)
   → 返回任务列表
```

### 示例 2：心跳保活（Agent 端定时任务）

**Agent 每 5 分钟定时执行：**
```bash
curl -X POST https://hevis.zwkjxm.com/api/v1/skill.heartbeat \
  -H "Content-Type: application/json" \
  -d '{
    "skill_key": "my-coze-assistant-v2",
    "platform": "coze",
    "platform_user_id": "coze_user_abc123"
  }'
```
**响应：**
```json
{
  "id": "sb_20260530120100",
  "skill_key": "my-coze-assistant-v2",
  "status": "active",
  "last_heartbeat": "2026-05-30T12:05:00Z",
  "created_at": "2026-05-30T12:01:00Z"
}
```

### 示例 3：能力域调用——搜索知识库

**用户输入：** "帮我查一下上次和王总的会议记录"

**Agent 执行：**
```bash
curl -X POST https://hevis.zwkjxm.com/api/v1/mcp \
  -H "Content-Type: application/json" \
  -H "X-Member-ID: member_uuid_xxx" \
  -d '{
    "tool": "hevis_memory_read",
    "params": {
      "query": "王总 会议 记录",
      "domain": "record"
    }
  }'
```

**期望输出：**
```json
{
  "results": [
    {
      "title": "与王总项目对接会",
      "date": "2026-05-28",
      "summary": "讨论了Q2项目进度、资源调配方案...",
      "tags": ["会议", "客户", "项目"]
    }
  ]
}
```

### 示例 4：管理员撤销连接

用户（管理员）在管理后台操作：
```bash
# 查看当前激活码列表
curl -H "Authorization: Bearer <token>" \
  https://hevis.zwkjxm.com/api/v1/member/codes

# 撤销一个旧的绑定
curl -X DELETE -H "Authorization: Bearer <token>" \
  https://hevis.zwkjxm.com/api/v1/member/bindings/old-agent-v1
```

**响应：**
```json
{ "message": "binding revoked" }
```

### 示例 5：Qoder/WorkBuddy MCP 配置

**下载的配置文件内容：**
```json
{
  "name": "hevis-os",
  "server_url": "https://hevis.zwkjxm.com/api/v1/mcp",
  "tools": [
    "hevis_memory_write",
    "hevis_memory_read",
    "hevis_memory_search",
    "hevis_knowledge_query"
  ],
  "authentication": {
    "type": "header",
    "header_name": "X-Member-ID",
    "header_value": "member_uuid_xxx"
  }
}
```

---

## 附录

### A. API 端点汇总

| 端点 | 方法 | 认证 | 用途 |
|---|---|---|---|
| `/api/v1/member/codes` | POST | Member | 生成激活码 |
| `/api/v1/member/codes` | GET | Member | 查看激活码列表 |
| `/api/v1/member/codes/:id` | DELETE | Member | 撤销激活码 |
| `/api/v1/skill.activate` | POST | Code | 第三方激活 |
| `/api/v1/skill.heartbeat` | POST | Binding | 心跳保活 |
| `/api/v1/skill.stale-bindings` | GET | Member | 查询失活绑定 |
| `/api/v1/member/bindings/:skill_key` | DELETE | Member | 撤销绑定 |
| `/api/v1/skill/config/:platform/:code` | GET | Code | 下载平台配置 |
| `/skill/install/:code` | GET | Code | 安装引导页 |

### B. 状态枚举

**SkillCode 状态：** `pending → bound | expired | revoked`
**Binding 状态：** `active → revoked`

### C. 平台标识

| 平台 | `platform` 参数值 | 配置文件格式 |
|---|---|---|
| Qoder/WorkBuddy | `qoder-workbuddy` 或 `qoder` | MCP JSON |
| Coze | `coze` | OpenAPI YAML |
| Dify | `dify` | Plugin YAML |
| ChatGPT | `chatgpt` | Actions JSON |
