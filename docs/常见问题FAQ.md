# 常见问题FAQ

## 脚本调试

### Q: 脚本在 Chrome 中正常工作，盒子上返回空？

A: 常见原因：
1. **页面有 Cloudflare 验证**：`cfAutoHandle` 默认 `true`，检测到挑战会自动弹出 WebView 供用户验证，脚本无需自己弹窗。注意**不要在挑战文档里执行 `evaluateJavascript`**（DOM 复核、轮询都不行），否则会打断 CF 采集信号，表现为「验证通过但仍返回 403」。用 `"logLevel": "debug"` 观察 `CF_ALLOW:` 日志确认是否进入了挑战流程。
2. **blockList 误伤**：检查 blockList 是否屏蔽了关键 JS 文件。尝试临时清空 blockList 对比测试。CSS/字体由 `blockCssAndFont` 独立处理，不要混在 blockList 里。
3. **执行时机**：省略 `@run-at` 时脚本在 `window load` 后才注入；如果页面是 SPA，数据可能仍未渲染。改用 `@run-at document-start` / `document-end`，或加 `setTimeout` 轮询等待。

### Q: 如何在 Chrome DevTools 中调试特定的爬虫方法？

A: 修改脚本中的 `GMSpiderArgs.fName` 和 `GMSpiderArgs.fArgs`，例如：
```javascript
GMSpiderArgs.fName = "categoryContent";
GMSpiderArgs.fArgs = ["type_id_value", "1", true, {}];  // (tid, pg, filter, extend)
```
然后刷新目标页面，Console 中即可看到返回结果。

### Q: 列表返回了很多重复视频？

A: 使用 `new Set()` 按 `vod_id` 去重：
```javascript
const seen = new Set();
$('a[href*="vod/"]').each(function () {
    const id = extractId(this.href);
    if (!id || seen.has(id)) return;
    seen.add(id);
    // ... 添加列表项
});
```

---

## 配置问题

### Q: blockList 的 `*.css*` 会不会影响页面渲染？

A: 不会。GM 爬虫使用无头 WebView，页面对用户不可见，CSS 样式完全不需要。屏蔽 CSS 可以减少大量网络请求，显著提升加载速度。

不过专用开关更省事：`blockCssAndFont`（默认 `true`）已经统一屏蔽 CSS 和字体，且挑战期间会自动放行；因此不要再把 `*.css*` / `*.woff2*` 写进 `blockList`。

### Q: `pagecount` 返回什么值最合适？

A: 如果存在"下一页"链接 → 返回 `9999`，让壳应用持续翻页直到返回空列表后自动停止。如果没有"下一页" → 返回当前页码或 `1`。

### Q: `playUrlMatch` 怎么用？

A: 当播放链接需要从网络请求中嗅探时（例如页面 JS 动态加载 m3u8 地址）：
1. 在 index.json 配置 `playUrlMatch: ["*.m3u8*"]`
2. 脚本播放源 `type` 设为 `"match"`
3. WebView 会自动捕获第一个匹配规则的请求 URL 替换播放地址

---

## 脚本问题

### Q: 页面自带的 `$` (jQuery) 和 `@require` 的 jQuery 冲突？

A: 如果页面自带 jQuery，不要在 `@require` 中再次引入。直接使用页面的 `$` 即可。在 Console 中执行 `typeof $` 确认。

### Q: 详情页 `vod_play_url` 格式是什么？

A: 这是 CatVod 的标准字段，但在 GM 爬虫里**推荐直接返回 `vod_play_data`**，不要手写 `vod_play_url`——GM 会把 `vod_play_data` 序列化并 base64 编码后自动写进 `vod_play_url`。

GM 生成的 `vod_play_url` 形态是 `分集标题$base64(PlayMedium JSON)`：
- 线路之间仍用 `$$$` 分隔（对应 `vod_play_from`）
- 分集之间用 `#` 分隔
- `$` 之后是 base64，不是裸 URL

兼容说明：脚本返回旧式 `分集标题$播放链接`（不含 base64）仍可识别，会被当作 `type: "direct"` 处理。但推荐用 `vod_play_data`，可显式控制 `type`、`ext.url`、`ext.header` 等字段。

### Q: 搜索页的分页参数在哪？

A: 搜索方法签名是 `searchContent(key, quick, pg)`，pg 在 `arguments[2]`。

### Q: 如何传递和接收筛选器参数？

A: `homeContent` 返回的 `filters` 中，用户选择的 `v` 值会自动传递到 `categoryContent` 的 `extend` 参数，同时也拼接到 `loadUrl` 模板的对应变量中。例如 `filter: { key: 'sort' }` → URL 中 `${sort}` 被替换为选中值。

---

## 环境问题

### Q: 盒子上看不到调试信息？

A: 三种方式：
1. 设置 `"logLevel": "debug"`，`adb logcat -s GMSpider` 可见 `MATCH:`、`BLOCK:`、`ALLOW:`、`CF_ALLOW:` 等详细日志
2. 设置 `"remoteDebug": true`，用 Chrome `chrome://inspect/#devices` 调试（仅开启调试并延迟销毁 WebView，**不会**让 WebView 变为可见）
3. 需要让 WebView 真正可见时，只能在脚本里调用 `GmSpiderInject.ShowWebview()`

### Q: 脚本太大导致加载慢？

A: 优化建议：
1. 保持 `blockNetworkImage: true`（默认即开）
2. 完善 `blockList` 屏蔽不必要的请求
3. 减少 `@require` 外部依赖，优先使用页面自带库
4. 移除 `console.log` 调试语句

### Q: 为什么选择了 `type: 3`？

A: CatVod 中 `type: 3` 表示使用自定义 Jar 爬虫（非内置解析器）。GM 爬虫属于此类。