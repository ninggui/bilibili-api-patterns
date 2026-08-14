---
name: bilibili-api-patterns
description: "B站 API: feed aggregation, cookie pitfalls, anti-412."
---

# B站 API 使用模式与避坑

## 核心原则：聚合优于遍历
避免逐个 UP 主 / 逐个视频遍历 B站 API，极易触发 412 风控。优先使用聚合接口。

## 关注动态聚合接口（替代逐个UP主爬取）

### 主力端点
```
GET https://api.bilibili.com/x/polymer/web-dynamic/v1/feed/all
```

| 参数 | 值 | 说明 |
|------|-----|------|
| `type` | `video` | 仅视频动态 (DYNAMIC_TYPE_AV)，推荐 |
| `type` | `all` | 全部动态 (AV+转发+图文+专栏) |
| `type` | `article` | 仅专栏文章 |
| `page` | 1, 2, ... | 页码 |
| `offset` | string | 翻页游标，取自上一页响应 `data.offset` |

- 每页 20 条，支持持续翻页（实测 20 页/400 条仍 `has_more=True`）
- `data.items[].modules.module_dynamic.major.archive` 包含: `bvid`, `aid`, `title`, `cover`, `jump_url`, `stat.play`, `stat.danmaku`, `duration_text`, `badge`
- **缺失 `pubdate`**：需额外调用 `/x/web-interface/view?bvid=xxx` 获取发布时间。这是主要代价，但仍远优于逐个 UP 主遍历（20 次 vs 1000+ 次 API 调用）

### 风控对比
| | 聚合接口 | 逐个UP主遍历 |
|---|---|---|
| 获取 400 条视频的 API 调用数 | ~20 次 | 620+400 = ~1020 次 |
| 412 风控风险 | 低（单次翻页，间隔可控） | 高（高频请求，极易触发） |

## Cookie 传递陷阱

**问题**：`requests.Session().cookies.set()` 会对 `%2C`（SESSDATA 中的逗号编码）做二次 URL 编解码，导致 B站服务端收到的 Cookie 值不正确，返回 `-101 账号未登录`。

**解决**：必须在 HTTP Header 中手动设置原始 Cookie 字符串，绕过 requests 的 Cookie jar：

```python
SESSDATA = '52819ffd%2C1801827161%2C...'  # 保持原始 %2C 编码
raw_cookie = f'SESSDATA={SESSDATA}; bili_jct={BILI_JCT}; buvid3={BUVID3}; DedeUserID={DedeUserID}'
headers = {
    'Cookie': raw_cookie,
    'User-Agent': 'Mozilla/5.0 ...',
    'Referer': 'https://www.bilibili.com/',
}
resp = requests.get(url, params=params, headers=headers)
```

## 已废弃的旧版接口（不要使用）

| 端点 | 状态 |
|------|------|
| `/x/feed/follow` | 404 已下线 |
| `/x/v2/feed/index` | 404 已下线 |

## SESSDATA 过期判断

- SDK `get_self_info()` 返回 `-101 账号未登录` 即 SESSDATA 过期
- 直接调用 feed API 也返回 `-101` 可确认
- SESSDATA 时间戳在 B站 cookie 中为 `%2C` 分隔的逗号格式，第二个字段为过期时间
- 不同来源的 SESSDATA 可能不同（如脚本中硬编码的 vs 任务描述中的），以实际可用的为准

## 验证方法

```python
from bilibili_api import Credential, user
cred = Credential(sessdata=SESSDATA, bili_jct=BILI_JCT, buvid3=BUVID3)
info = await user.get_self_info(cred)
# 成功返回: {'name': '...', 'mid': '...'}
# 失败抛出: -101 账号未登录
```

## 参考
- 实测数据: `references/feed-api-test-results.md`（本次测试完整结果）
- **浏览器 Console 兜底**: `references/browser-console-fallback.md`（terminal 被 consent 拦 / space API 需 WBI 签名时，用 browser+console fetch 研究 B站 KOL，2026-08 验证）
- 相关 skill: `bilibili-watch-later`（需 `hermes curator adopt bilibili-watch-later` 后可合并）
- 脚本位置: `/path/to/data/skills/productivity/bilibili-watch-later/scripts/bilibili_watch_later.py`
- 当前有效凭证: SESSDATA=`52819ffd...`, UID=228407179, 用户名=互联网实习生
