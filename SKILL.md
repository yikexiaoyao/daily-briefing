---
name: daily-briefing
description: 定时汇报每日/每周任务日程，支持 iCal/ICS 订阅源，配置简单，开箱即用。
---

# Daily Briefing Skill

定时汇报每日/每周任务日程。从 iCal/ICS 订阅源读取数据，生成结构化简报。

## 快速部署

### 第一步：准备信息

部署前需要收集 5 个配置项，按需填写：

| 配置项 | 说明 | 必填 | 示例 |
|--------|------|------|------|
| `ICAL_URL` | 日历/任务的 iCal 订阅链接 | ✅ 是 | `https://ticktick.com/...` |
| `TIMEZONE` | 时区 | ✅ 是 | `Asia/Shanghai` |
| `DELIVERY_TO` | 推送目标 | ✅ 是 | `telegram:123456789` |
| `DELIVERY_CHANNEL` | 推送渠道 | ✅ 是 | `telegram` |
| `DELIVERY_ACCOUNT` | 推送账号 | 否 | `default` |

### 第二步：获取 iCal 链接

**滴答清单 / TickTick：**
1. 打开滴答清单 → 设置 → 日历订阅
2. 复制链接（`webcal://` 开头的话，把 `webcal` 改成 `https`）

**Google Calendar：**
1. 打开 Google Calendar → 设置 → 集成日历
2. 找到"私享网址（iCal 格式）"，复制链接

**Apple Calendar（iCloud）：**
1. 打开 iCloud.com → 日历 → 共享日历
2. 启用"公共日历"，复制订阅链接

### 第三步：确认推送配置

**Telegram 私聊：**
- `DELIVERY_TO`: `telegram:你的用户ID`（可从会话信息中获取）
- `DELIVERY_CHANNEL`: `telegram`
- `DELIVERY_ACCOUNT`: `default`

**Telegram 群组：**
- `DELIVERY_TO`: `telegram:群组ID`
- `DELIVERY_CHANNEL`: `telegram`
- `threadId`: `话题ID`（可选，用于话题群）

### 第四步：创建任务

将配置项替换到下方三个 JSON 中的占位符，然后逐一用 `cron` 工具的 `action: "add"` 创建：
- `<ICAL_URL>` → 你的 iCal 链接
- `<TIMEZONE>` → 你的时区
- `<DELIVERY_TO>` → 你的推送目标
- `<DELIVERY_CHANNEL>` → 你的推送渠道
- `<DELIVERY_ACCOUNT>` → 你的推送账号

### 第五步：测试

创建后手动触发一次，确认输出格式：
```
cron action: "run"
jobId: <创建后返回的 jobId>
runMode: "force"
```

---

## 任务定义

### 晨报（每天 8:00）

```json
{
  "name": "每日晨报",
  "schedule": { "kind": "cron", "expr": "0 8 * * *", "tz": "<TIMEZONE>" },
  "payload": {
    "kind": "agentTurn",
    "message": "现在是早上8点，执行每日晨报任务。\n\n步骤：\n1. 用 web_fetch 获取 iCal 订阅：<ICAL_URL>\n2. 解析 iCal 内容，筛选出今天的待办事项\n3. 整理待办事项，标注截止时间\n\n汇报要求：\n- 有内容就整理罗列，标注截止时间\n- 没有内容就一句话简要说明，不要写\"无\"、\"暂无\"等字样，不要留空模板或占位符\n- 语气简洁自然，不要生硬，不要铺垫废话，不要添加多余的评价或感慨\n- 只输出实际存在的内容，不编造、不推测\n- 标题格式：☀️ 晨报 ｜ X月X日 周X"
  },
  "sessionTarget": "isolated",
  "delivery": { "mode": "announce", "to": "<DELIVERY_TO>", "channel": "<DELIVERY_CHANNEL>", "accountId": "<DELIVERY_ACCOUNT>" },
  "enabled": true
}
```

### 晚报（每天 20:00）

```json
{
  "name": "每日晚报",
  "schedule": { "kind": "cron", "expr": "0 20 * * *", "tz": "<TIMEZONE>" },
  "payload": {
    "kind": "agentTurn",
    "message": "现在是晚上8点，执行每日晚报任务。\n\n步骤：\n1. 用 web_fetch 获取 iCal 订阅：<ICAL_URL>\n2. 解析 iCal 内容，列出今天的所有条目\n\n汇报要求：\n- 有内容就整理罗列，每条标注截止时间\n- 没有内容就一句话简要说明，不要写\"无\"、\"暂无\"等字样，不要留空模板或占位符\n- 语气简洁自然，不要生硬，不要铺垫废话，不要添加多余的评价或感慨\n- 只输出实际存在的内容，不编造、不推测\n- 标题格式：🌃 晚报 ｜ X月X日 周X"
  },
  "sessionTarget": "isolated",
  "delivery": { "mode": "announce", "to": "<DELIVERY_TO>", "channel": "<DELIVERY_CHANNEL>", "accountId": "<DELIVERY_ACCOUNT>" },
  "enabled": true
}
```

### 周报（每周日 22:00）

```json
{
  "name": "每周周报",
  "schedule": { "kind": "cron", "expr": "0 22 * * 0", "tz": "<TIMEZONE>" },
  "payload": {
    "kind": "agentTurn",
    "message": "现在是周日晚上10点，执行每周晚报任务。\n\n步骤：\n1. 用 web_fetch 获取 iCal 订阅：<ICAL_URL>\n2. 解析 iCal 内容，筛选出本周一到周日的所有条目\n3. 整理汇总本周事项\n\n汇报要求：\n- 有内容就整理罗列，每条标注截止时间\n- 没有内容就一句话简要说明，不要写\"无\"、\"暂无\"等字样，不要留空模板或占位符\n- 语气简洁自然，不要生硬，不要铺垫废话，不要添加多余的评价或感慨\n- 只输出实际存在的内容，不编造、不推测\n- 标题格式：📊 周报 ｜ X月X日 - X月X日"
  },
  "sessionTarget": "isolated",
  "delivery": { "mode": "announce", "to": "<DELIVERY_TO>", "channel": "<DELIVERY_CHANNEL>", "accountId": "<DELIVERY_ACCOUNT>" },
  "enabled": true
}
```

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

### 修改语言

把 `payload.message` 中的中文提示翻译为其他语言即可。

---

## 注意事项

- **iCal 是只读的**：只能读取条目，无法判断完成状态或修改条目
- **不编造内容**：如果某时段没有条目，简要说明即可，不写"无"、不留空模板
- **格式统一**：三个汇报的输出风格一致 — 标题 → 条目，没有多余板块
- **仅依赖 `web_fetch` + `cron`**：无需额外工具或依赖，任何支持这两个工具的环境都能用
