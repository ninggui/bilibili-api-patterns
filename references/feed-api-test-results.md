# B站 Feed API 实测数据 (2026-08-09)

## 测试环境
- Cookie: SESSDATA=52819ffd... (UID=228407179, 用户名=互联网实习生)
- bilibili-api-python v17.4.2
- Python 3.13.5, requests

## 测试结果

### 1. 聚合接口测试

| 接口 | type | 返回码 | 条目数 | has_more | 动态类型分布 |
|------|------|--------|--------|----------|-------------|
| feed/all | video | 0 | 20 | True | DYNAMIC_TYPE_AV: 20 |
| feed/all | all | 0 | 20 | True | AV:15, FORWARD:3, DRAW:2 |
| feed/all | live | 0 | 20 | True | AV:15, FORWARD:3, DRAW:2 (与all相同，非纯直播) |
| feed/all | article | 0 | 20 | True | DYNAMIC_TYPE_ARTICLE: 20 |
| feed/space | video | 0 | 12 | - | - |

### 2. 翻页测试

翻 20 页获取 400 条视频动态，仍有 `has_more=True`。
每页 offset 递减（如 1234507702665740337 → 1234149635533045769）。

### 3. archive 数据结构

```json
{
  "type": 1,
  "bvid": "BV1JQu264EaE",
  "aid": "117065339963919",
  "cover": "http://i1.hdslb.com/bfs/archive/...jpg",
  "jump_url": "//www.bilibili.com/video/BV1JQu264EaE",
  "stat": {"danmaku": "1", "play": "1102", "vt": ""},
  "duration_text": "00:59",
  "title": "末日将至！OpenAI 史诗巨物 Doug 模型定档...",
  "desc": "",
  "badge": {"text": "投稿视频", "bg_color": "#FB7299"},
  "enable_vt": 0,
  "disable_preview": 0,
  "premiere_online": "",
  "stat_hidden": 0
}
```

### 4. 缺失字段

- `pubdate` / `ctime`: 不在此接口返回，需额外调 `/x/web-interface/view?bvid=xxx`
- `duration` (秒数): 不在此接口返回，仅有 `duration_text` (如 "00:59")
- `favorite` stat: None

### 5. Cookie 调试过程

1. `requests.Session().cookies.set('SESSDATA', ...)` → `-101 账号未登录`（requests 对 %2C 做了二次编解码）
2. `requests.Session()` + 手动 `Cookie` header → 同样 `-101`（早期测试用了已过期的 SESSDATA=b79ec5b4...）
3. 切换到脚本中的 SESSDATA=52819ffd... → 成功
4. 结论：必须 (a) 使用有效的 SESSDATA (b) 手动设置 Cookie header 原始字符串

### 6. 关注列表

- 关注总数: 620 位 UP 主
- 逐个遍历风险: 620 次 API + 每个视频查详情 = 1000+ 次 → 极高风控风险
- 聚合方案: ~20 次翻页 → 低风险
