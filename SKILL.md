---
name: openclaw-daily-briefing
description: 定时汇报每日/每周任务日程，通过 dida CLI 直连滴答清单 API，支持优先级分组、已完成/未完成分类展示。触发场景：晨报、晚报、周报、今日待办、任务汇报、查看任务、任务日程。
metadata:
  openclaw:
    emoji: "📅"
    always: false
    requires:
      bins: ["node"]
      env: []
triggers:
  - 晨报
  - 晚报
  - 周报
  - 今日待办
  - 任务汇报
  - 查看任务
  - 任务日程
  - daily briefing
  - morning report
  - evening report
  - weekly report
---

# Daily Briefing Skill

定时汇报每日/每周任务日程。通过 `dida` CLI（@suibiji/dida-cli）直连滴答清单 API，获取实时任务数据。

---

## 前置条件

| 序号 | 项目 | 要求 | 检查方式 |
|------|------|------|----------|
| 1 | OpenClaw | >= v2026.6.0 | `openclaw --version` |
| 2 | Node.js | >= 18 | `node --version` |
| 3 | dida CLI | @suibiji/dida-cli 已安装 | `dida --version` |
| 4 | OAuth 授权 | 已执行登录 | `dida auth login` |
| 5 | 推送渠道 | Telegram / 其他渠道已配置 | 确认 DELIVERY_TO 等变量 |

---

## 数据源

| 序号 | 命令 | 用途 |
|------|------|------|
| 1 | `dida task filter --status 0 --json` | 获取所有未完成任务（JSON） |
| 2 | `dida task completed` | 获取已完成任务 |

---

## 输出格式规范

### 晨报输出

```
☀️ 晨报｜6月28日 周六

🟡 今日待办
  • [中] 14:00 产品评审会
  • [低] 整理本周笔记

⏳ 待跟进
  • [高] 提交季度报告（已逾期）
  • [中] 周五前完成设计稿
```

> 注：只展示有内容的分区，空分区直接省略不显示。

### 晚报输出

```
🌃 晚报｜6月28日 周六

✅ 今日已完成
  • [中] 完成 API 接口文档
  • [低] 回复客户邮件

⏳ 今日未完成
  • [高] 提交季度报告（已逾期）
  • [中] 14:00 产品评审会（已取消）
```

> 注：只展示有内容的分区，空分区直接省略不显示。

### 周报输出

```
📊 周报｜6月23日 - 6月29日

✅ 本周完成
  • [高] 完成用户系统重构
  • [中] 修复 3 个线上 Bug
  • [低] 整理技术文档

⏳ 未完成/待跟进
  • [高] 支付模块开发（进度 60%）
  • [中] 性能优化方案（待评审）
```

> 注：只展示有内容的分区，空分区直接省略不显示。

通用格式规则（三个汇报统一遵守）：
- 每条必须带优先级标签 `[高/中/低/无]`
- 日期中间不加空格（如 `6月29日` 不是 `6 月 29 日`）
- 空分区直接省略，不显示"无"、"暂无"或"(无)"
- 所有分区都没内容 → 一句话简要说明
- 不输出多余的解释、评价、感慨或模板文字

分区说明：
- 🟡 今日待办 — 今天截止的任务
- ⏳ 待跟进 — 逾期或未来需要跟进的任务
- ✅ 今日已完成 — 今日完成的任务（晚报）
- ⏳ 今日未完成 — 今日截止但未完成的任务（晚报）
- ✅ 本周完成 — 本周完成的任务（周报）
- ⏳ 未完成/待跟进 — 需跟进的任务（周报）

---

## 快速开始

```bash
# 1. 安装 dida CLI
npm i -g @suibiji/dida-cli

# 2. 登录授权
dida auth login

# 3. 测试获取任务
dida task filter --status 0 --json

# 4. 安装技能到本地
openclaw skills install openclaw-daily-briefing

# 5. 配置推送渠道并创建 cron 任务（见下方部署引导）
```

---

## 安装与部署

### 方式 A：从 ClawHub 安装（推荐）

```bash
# 搜索技能
openclaw skills search openclaw-daily-briefing

# 安装
openclaw skills install openclaw-daily-briefing

# 验证
openclaw skills list | grep daily-briefing
```

### 方式 B：本地部署

```bash
# 1. 创建技能目录
mkdir -p ~/.openclaw/workspace/skills/openclaw-daily-briefing

# 2. 将 SKILL.md 放入目录
cp SKILL.md ~/.openclaw/workspace/skills/openclaw-daily-briefing/

# 3. 验证安装
openclaw skills list | grep daily-briefing
```

### 安装后验证

| 检查项 | 命令 | 预期结果 |
|--------|------|----------|
| 技能状态 | `openclaw skills list` | `✓ ready │ 📅 openclaw-daily-briefing` |
| dida 登录 | `dida auth status` | 已授权 |
| 数据获取 | `dida task filter --status 0 --json` | 返回任务 JSON |

---

## 快速部署：创建 Cron 任务

### 确认推送配置

**Telegram 私聊：**
- `DELIVERY_TO`: `telegram:你的用户ID`
- `DELIVERY_CHANNEL`: `telegram`
- `DELIVERY_ACCOUNT`: `default`

### 创建任务

逐一用 `cron` 工具的 `action: "add"` 创建三个任务：

---

## 晨报（每天 8:00）

```json
{
  "name": "每日晨报 - 任务日程提醒",
  "schedule": { "kind": "cron", "expr": "0 8 * * *", "tz": "Asia/Shanghai" },
  "payload": {
    "kind": "agentTurn",
    "lightContext": true,
    "timeoutSeconds": 600,
    "message": "执行每日晨报任务。\n\n步骤：\n1. 执行 `dida task filter --status 0 --json` 获取所有未完成任务（JSON 格式）\n2. 从结果中按 due-date 分类整理\n\n汇报要求（严格遵守）：\n- **只输出最终结果**，不要输出任何分析、思考、推理过程\n- 标题格式：☀️ 晨报｜X月X日 周X\n- 按优先级分组展示：🔴 逾期 → 🟡 今日 → 🟢 近期 → 📋 无截止日期\n- 每条标注优先级（高/中/低/无）\n- 没有内容就一句话简要说明，不要写\"无\"、\"暂无\"等字样，不要留空模板或占位符\n- **不要有任何额外的解释、评价、感慨或模板文字**"
  },
  "sessionTarget": "isolated",
  "delivery": { "mode": "announce", "to": "<DELIVERY_TO>", "channel": "<DELIVERY_CHANNEL>", "accountId": "<DELIVERY_ACCOUNT>", "bestEffort": true },
  "enabled": true
}
```

## 晚报（每天 20:00）

```json
{
  "name": "每日晚报 - 任务日程汇总",
  "schedule": { "kind": "cron", "expr": "0 20 * * *", "tz": "Asia/Shanghai" },
  "payload": {
    "kind": "agentTurn",
    "timeoutSeconds": 600,
    "message": "执行每日晚报任务。\n\n步骤：\n1. 执行 `dida task completed` 获取已完成任务，筛选今天的\n2. 执行 `dida task filter --status 0 --json` 获取未完成任务，筛选今天截止但尚未完成的\n\n汇报要求：\n- 标题格式：🌃 晚报｜X月X日 周X\n- 分两部分：✅ 今日已完成 → ⏳ 今日未完成\n- 没有内容就一句话简要说明，不要写\"无\"、\"暂无\"等字样，不要留空模板或占位符\n- 语气简洁自然，不要铺垫废话，不要添加多余的评价或感慨\n- 只输出实际存在的内容，不编造、不推测"
  },
  "sessionTarget": "isolated",
  "delivery": { "mode": "announce", "to": "<DELIVERY_TO>", "channel": "<DELIVERY_CHANNEL>", "accountId": "<DELIVERY_ACCOUNT>", "bestEffort": true },
  "enabled": true
}
```

## 周报（每周日 20:30）

```json
{
  "name": "每周晚报 - 周任务汇总",
  "schedule": { "kind": "cron", "expr": "30 20 * * 0", "tz": "Asia/Shanghai" },
  "payload": {
    "kind": "agentTurn",
    "timeoutSeconds": 600,
    "message": "执行每周周报任务。\n\n步骤：\n1. 执行 `dida task completed` 获取已完成任务，筛选本周（周一到周日）的\n2. 执行 `dida task filter --status 0 --json` 获取未完成任务\n\n汇报要求：\n- 标题格式：📊 周报｜X月X日 - X月X日\n- 分两部分：✅ 本周完成 → ⏳ 未完成/待跟进\n- 没有内容就一句话简要说明，不要写\"无\"、\"暂无\"等字样，不要留空模板或占位符\n- 语气简洁自然，不要铺垫废话，不要添加多余的评价或感慨\n- 只输出实际存在的内容，不编造、不推测"
  },
  "sessionTarget": "isolated",
  "delivery": { "mode": "announce", "to": "<DELIVERY_TO>", "channel": "<DELIVERY_CHANNEL>", "accountId": "<DELIVERY_ACCOUNT>", "bestEffort": true },
  "enabled": true
}
```

---

## 常见用例

| 序号 | 场景 | 触发方式 | 说明 |
|------|------|---------|------|
| 1 | 查看今日待办 | "今日待办" / "今天有什么任务" | 实时获取未完成任务 |
| 2 | 手动触发晨报 | "晨报" / "今日任务汇报" | 按晨报格式输出 |
| 3 | 查看已完成 | "今天完成了什么" | 筛选今日已完成任务 |
| 4 | 查看本周进度 | "本周任务汇报" | 按周报格式汇总 |
| 5 | 自动推送 | cron 定时任务 | 晨报/晚报/周报自动推送 |

---

## 自定义

### 修改时间

修改 `expr` 字段即可：

| 场景 | Cron 表达式 |
|------|------------|
| 工作日 7:30 | `"30 7 * * 1-5"` |
| 每天两次（7:00 和 19:00） | `"0 7,19 * * *"` |
| 每 6 小时 | `"0 */6 * * *"` |
| 周一和周五 9:00 | `"0 9 * * 1,5"` |

### 禁用某个汇报

将 `"enabled": false` 即可暂停，无需删除。

---

## 更新与卸载

```bash
# 从 ClawHub 更新
openclaw skills update openclaw-daily-briefing

# 从 ClawHub 卸载
openclaw skills uninstall openclaw-daily-briefing

# 或手动删除目录
rm -rf ~/.openclaw/workspace/skills/openclaw-daily-briefing
```

---

## 注意事项

- **dida CLI 需要登录态**：token 存储在本地，过期需重新 `dida auth login`
- **不编造内容**：如果某时段没有条目，简要说明即可
- **格式统一**：三个汇报的输出风格一致
- **仅依赖 `dida` CLI + `cron`**：无需 iCal 订阅链接
