# bilibili-api-patterns

![GitHub stars](https://img.shields.io/github/stars/ninggui/bilibili-api-patterns)
![License](https://img.shields.io/github/license/ninggui/bilibili-api-patterns)
[![SkillHub](https://img.shields.io/badge/SkillHub-在线安装-blue)](https://skillhub.cn/skills/bilibili-api-patterns)

B站 API 使用模式合集：feed 聚合、cookie 陷阱、防 412 反爬。来自真实业务场景沉淀，直接可用的 Agent 技能。

## 快速使用

```bash
# 技能放入 Agent skills 目录后，直接触发：
"拉取我关注UP主的最新视频"        # feed 聚合
"检查 B站 412 风控原因"           # 避坑诊断
"把 UP主新视频加入稍后再看"       # 自动任务
```

## 核心能力

| 主题 | 要点 |
|------|------|
| Feed 聚合 | 关注流/推荐流 API 路径与参数、分页去重 |
| Cookie 陷阱 | 哪些接口需要 cookie、缺失时的具体表现 |
| 防 412 | 请求频率、UA、referer 的正确姿势 |
| 稍后再看 | 批量添加、去重、24h 累积策略 |

## 避坑清单（已实测）

- 24 小时模式：每 UP 主间隔 3 秒 → 50 个 UP 主约 2.5 分钟可通过
- 30 天模式会触发 412 风控，不可用
- 固定时间窗（如每天 18:00）自动跑累积覆盖，避免高频短跑

## 安装

- **Hermes**: `skills/` 目录
- **Claude**: `~/.claude/skills/`
- 或 SkillHub 一键安装：https://skillhub.cn/skills/bilibili-api-patterns

## 内容结构

- `SKILL.md` — 核心技能定义（触发条件、执行流程、避坑清单）
- `references/` — 参考文件

## 许可

MIT
