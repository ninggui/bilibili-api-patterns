# bilibili-api-patterns

B站 API 使用模式合集：feed 聚合、cookie 陷阱、防 412 反爬。适用于 B站相关自动化（关注UP主监控、视频列表、稍后再看等）。

## 这是什么

一个可复用的 AI Agent 技能（Skill），来自真实业务场景沉淀。记录 B站 API 调用中的关键模式与避坑经验，避免重复踩坑。

## 解决的问题

- B站 API 各种返回格式记不住
- cookie 过期/缺失导致 412 反爬
- feed 流分页、排序规则不明确
- 频繁调用被风控

## 核心知识点

| 主题 | 要点 |
|------|------|
| Feed 聚合 | 关注流/推荐流的 API 路径与参数 |
| Cookie 陷阱 | 哪些接口需要 cookie，缺失时的表现 |
| 防 412 | 请求频率、UA、referer 注意事项 |
| 分页 | page_size/page_num 规则，防重复 |

## 使用方式

将本仓库内容放入你的 Agent 技能目录：

- **Hermes**: `skills/` 目录
- **Claude**: `~/.claude/skills/`
- **其他 Agent**: 按对应 SKILL.md 格式

Agent 会在匹配触发条件时自动加载并使用。

## 典型场景

- 每天定时拉取关注UP主的新视频
- 批量获取视频信息做分析
- 自动添加稍后再看

## 内容结构

- `SKILL.md` — 核心技能定义（触发条件、执行流程、避坑清单）
- `references/` — 可选参考文件

## 许可

MIT
