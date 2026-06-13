# Hevis-OS 官方 Skill

Hevis-OS 认知操作系统对外分发的标准化智能体能力包，遵循 **「好 Skill 的 8 层架构」** 标准编写。

## 快速概览

| 元信息 | 值 |
|---|---|
| Skill ID | `hevis-os-skill` |
| 版本 | `1.2.0` |
| 能力域 | 10 个（人/事/看/备/存/搜/记/摄/连/标） |
| 支持平台 | Qoder、WorkBuddy、Coze、Dify、ChatGPT |
| SPC 引用 | SPC §6 — 连（Connect）能力域 |
| 连接器版本 | ≥ `1.0.0` |

## 能力域

| 图标 | 域 | ID | 说明 | APIs |
|---|---|---|---|---|
| 👤 | 人 | person | 人物卡片、关系网络、评分管理 | memory.read, memory.write |
| 📋 | 事 | task | 项目、商机、任务、供应商管理 | memory.read, memory.write, workflow.execute |
| 📊 | 看 | brief | 每日简报、使用分析、运营数据 | memory.read, context.inject |
| 🎯 | 备 | prepare | 会前准备、材料整理、会议支持 | workflow.bootstrap, workflow.plan, workflow.execute |
| 📁 | 存 | store | 文档结构化、知识库管理、归档 | memory.write, memory.read |
| 🔍 | 搜 | search | 全网知识搜索、知识库检索 | memory.read |
| 🎙️ | 记 | record | 录音、会议记录、AI 分析、纪要生成 | memory.write, workflow.execute |
| 📥 | 摄 | ingest | 数据摄入：系统分享、相机、扫描、剪贴板、文件 | memory.write |
| 🔗 | 连 | connect | 第三方平台连接、Skill 码管理、外部 Agent 接入 | memory.read, memory.write |
| 🎯 | 标 | target | OKR 目标管理、目标网络、关键成果追踪 | memory.read, memory.write |

## 接入流程

### 第 1 步：生成 Skill Code

在 Hevis-OS 管理后台或 Android App 中生成激活码：

```
POST /api/v1/member/codes
→ { "code": "mcode_a1b2c3d4e5f6a7b8c9d0e1f2", "status": "pending" }
```

Code 有效期 **15 分钟**，超时自动失效。

### 第 2 步：打开安装引导页

```
https://hevis.zwkjxm.com/skill/install/<你的 Skill Code>
```

页面显示 4 个平台的配置下载入口。

### 第 3 步：下载平台配置文件

| 平台 | 配置文件 | 格式 |
|---|---|---|
| Qoder/WorkBuddy | `hevis-mcp-config.json` | MCP JSON |
| Coze | `hevis-coze-openapi.yaml` | OpenAPI YAML |
| Dify | `hevis-dify-plugin.yaml` | Plugin YAML |
| ChatGPT | `hevis-chatgpt-action.json` | Actions JSON |

### 第 4 步：在目标平台导入配置

- **Qoder/WorkBuddy**: 导入 MCP JSON 作为 MCP Server
- **Coze**: 导入 OpenAPI YAML 作为插件
- **Dify**: 导入 Plugin YAML 作为工具
- **ChatGPT**: 导入 Actions JSON 作为 GPTs Action

### 第 5 步：激活连接

Agent 启动后自动调用激活 API：

```
POST /api/v1/skill.activate
{
  "code": "mcode_a1b2c3d4e5f6a7b8c9d0e1f2",
  "platform": "coze",
  "platform_user_id": "coze_user_xxx",
  "skill_key": "my-coze-bot"
}
```

### 第 6 步：保持心跳

Agent 每 5-10 分钟调用一次心跳 API：

```
POST /api/v1/skill.heartbeat
{
  "skill_key": "my-coze-bot",
  "platform": "coze",
  "platform_user_id": "coze_user_xxx"
}
```

超过 15 分钟无心跳，绑定将标记为失活（stale）。

## 连接器管理

管理员可在管理后台进行连接生命周期管理：

- **查看激活码**：`GET /api/v1/member/codes`
- **撤销激活码**：`DELETE /api/v1/member/codes/:id`
- **撤销绑定**：`DELETE /api/v1/member/bindings/:skill_key`
- **查询失活**：`GET /api/v1/skill.stale-bindings`

## 文件结构

```
hevis-os-skill/
├── SKILL.md              # 8 层架构核心文件（身份 → 定义 → 场景 → 输入 → 输出 → 工作流 → 规则 → 示例）
├── capabilities.yaml     # 能力声明（含 10 域完整 name/description/icon/categories 字段）
├── examples/             # 接入与使用示例
├── scripts/              # 自动化脚本
├── resources/            # 配置模板与资源
├── tests/                # 测试用例与评估
├── README.md             # 本文件：快速开始与概览
└── .github/workflows/    # Release 通知 → 服务器同步
```

## 版本管理

版本通过 GitHub Release 管理。每发布一个新版本：

1. 创建 GitHub Release（tag）
2. Actions 工作流自动发送 Webhook 到服务器
3. 服务器更新 `capabilities.yaml` 元数据并刷新缓存

安装入口：`https://hevis.zwkjxm.com/skill/install/<Code>`

## 相关资源

- [SKILL.md](./SKILL.md) — 8 层架构完整说明
- [capabilities.yaml](./capabilities.yaml) — 机器可读能力声明
- [SPC §6 规范](https://hevis.zwkjxm.com/docs/spc#section6) — 连能力域定义
