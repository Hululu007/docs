# 常见问题FAQ

## 脚本调试

### Q: 脚本在 Chrome 中正常工作，盒子上返回空？

A: 常见原因：
1. **页面有 Cloudflare 验证**：盒子 WebView 可能被拦截。检查页面是否有 `#cf-wrapper` 元素，在脚本入口处检测并 toast 提示。
2. **blockList 误伤**：检查 blockList 是否屏蔽了关键 JS 文件。尝试临时清空 blockList 对比测试。
3. **`$(function(){})` 触发时机**：如果页面是 SPA（单页应用），DOM 就绪时数据可能还未渲染。尝试加 `setTimeout` 延迟执行。

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

A: 格式为 `分集标题$播放链接#分集标题2$播放链接2`。
- 多集用 `#` 分隔
- 标题和链接用 `$` 分隔
- 单集示例：`默认线路$https://example.com/video.m3u8`

多条播放线路通过 `$$$` 分隔（对应 `vod_play_from` 中的多个来源）。

### Q: 搜索页的分页参数在哪？

A: 搜索方法签名是 `searchContent(key, quick, pg)`，pg 在 `arguments[2]`。

### Q: 如何传递和接收筛选器参数？

A: `homeContent` 返回的 `filters` 中，用户选择的 `v` 值会自动传递到 `categoryContent` 的 `extend` 参数，同时也拼接到 `loadUrl` 模板的对应变量中。例如 `filter: { key: 'sort' }` → URL 中 `${sort}` 被替换为选中值。

---

## 环境问题

### Q: 盒子上看不到调试信息？

A: 三种方式：
1. 设置 `"debug": true`，WebView 变为可见，可观察加载状态
2. Chrome `chrome://inspect/#devices` 远程调试
3. `adb logcat -s GMSpider` 查看 Java 层日志

### Q: 脚本太大导致加载慢？

A: 优化建议：
1. 开启 `blockNetworkImage: true`
2. 完善 `blockList` 屏蔽不必要的请求
3. 减少 `@require` 外部依赖，优先使用页面自带库
4. 移除 `console.log` 调试语句

### Q: 为什么选择了 `type: 3`？

A: CatVod 中 `type: 3` 表示使用自定义 Jar 爬虫（非内置解析器）。GM 爬虫属于此类。