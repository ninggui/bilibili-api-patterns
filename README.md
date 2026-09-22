<div align="center">

# bilibili-api-patterns

**B站 API 避坑手册：聚合优于遍历，否则 412 风控教你做人。**

<p>
  <a href="#"><img src="https://img.shields.io/badge/wbi-signed-blue?logo=bilibili&logoColor=white" alt="WBI" /></a>
  <a href="#"><img src="https://img.shields.io/badge/avoid-412-red" alt="No 412" /></a>
</p>

[核心原则](#核心原则) · [主力接口](#主力接口) · [避坑](#避坑)

</div>

---

## 核心原则

**聚合优于遍历**。逐个 UP 主/逐个视频爬 = 必死。优先用聚合接口一次拉全。

## 主力接口

```
GET https://api.bilibili.com/x/polymer/web-dynamic/v1/feed/all
```

| 参数 | 说明 |
|------|------|
| type=video | 仅视频动态（推荐） |
| type=all | 全部动态 |
| page / offset | 翻页游标 |

## 避坑

- 未签名的 `x/player/v2` 字幕接口会串台，必须用 `x/player/wbi/v2` + wbi 签名
- 请求间隔 ≥ 2s，412 退避 10/20/30s
- cookie 里的 SESSDATA 是核心，bili_jct 用于写操作

## License

MIT
