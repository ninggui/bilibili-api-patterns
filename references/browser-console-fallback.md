# 浏览器 Console 兜底：B站 KOL 研究（2026-08-14 验证）

**场景**：terminal `curl`/`requests` 直连 `api.bilibili.com` 被审批拦截，或 `/x/space/arc/search` 返回 HTML（DOCTYPE）而非 JSON（space 系列接口需 WBI 签名，匿名直连拿不到数据）。此时用 browser 工具 + console fetch 兜底。

## 可用端点（浏览器上下文 + Referer 头即可）
```js
// 1. 视频详情（title/desc/stat）— 只需 Referer，无需 WBI
fetch('https://api.bilibili.com/x/web-interface/view?bvid=BVxxx',
  {headers: {'Referer': 'https://www.bilibili.com'}}).then(r => r.json())
// 2. 按关键词搜 UP主视频（翻页+按发布时间排序）
fetch('https://api.bilibili.com/x/web-interface/search/type?search_type=video&keyword=' +
  encodeURIComponent('UP主昵称') + '&page=1&order=pubdate',
  {headers: {'Referer': 'https://search.bilibili.com'}}).then(r => r.json())
```

## UP主视频列表提取流程（研究某 KOL 时）
1. `browser_navigate` 到 `https://search.bilibili.com/all?keyword=UP主昵称&order=pubdate`——**搜索页能渲染视频卡片**，即使 space 页显示"空间主人还没投过视频"（space SPA 未登录常加载失败）
2. 从 DOM 提取 BV：`document.querySelectorAll('a[href*="/video/BV"]')`（注意过滤推荐区卡片，按标题关键词匹配目标）
3. 逐 BV 调 view API 拿 title/desc/stat
4. 需要标题图谱时用 search/type 端点翻 6-8 页聚合（每页约 10 条，返回 title/play/pubdate）

## 避坑
- **别点卡片跳转**：搜索页点视频卡会在当前 tab 打开视频页，丢失搜索上下文——先提取 href 里的 BV，再直接调 API
- 搜索结果卡片标题可能含 HTML 标签，需 `title.replace(/<[^>]+>/g, '')`
- 视频笔记/无简介视频 `desc` 常为空字符串或 `"-"`——内容在口播里，标题+播放量已能提炼主题图谱
- 短链确认账号用 `curl -L -o /dev/null -w "%{url_effective}"`，能拿到带新 xsec_token 的完整主页 URL
