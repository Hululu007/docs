# 参考资料及Javascript类库推荐

## 本项目相关

| 资源 | 说明 |
| :--- | :--- |
| [SFW-Spiders](https://github.com/CatVodSpider-GM/SFW-Spiders) | 普通站点脚本合集 |
| [NSFW-Spiders](https://github.com/CatVodSpider-GM/NSFW-Spiders) | NSFW 站点脚本合集，含大量真实站点实现 |
| [CatVodSpider-WebMonkey](https://github.com/CatVodSpider-GM/CatVodSpider-WebMonkey) | 本爬虫源码 |
| [jQuery 模板](../templates/userScripts/jQuery.user.js) | 页面直接返回 HTML 时的起点 |
| [ajaxHook 模板](../templates/userScripts/ajaxHook.user.js) | 数据由 ajax 异步加载时的起点 |
| [Spiders-Lib/utils.js](https://cdn.jsdelivr.net/gh/CatVodSpider-GM/NSFW-Spiders@main/Spiders-Lib/utils.js) | 共用工具：`extract` / `extractDate` / `extractCode` / `hookFetch` 等，可直接 `@require` |

## Userscript 运行时

| 资源 | 说明 |
| :--- | :--- |
| [Android-WebMonkey](https://github.com/warren-bank/Android-WebMonkey) | 本爬虫内置的 Greasemonkey 运行时，支持 `GM_getValue`、`GM_xmlhttpRequest` 等 API |
| [Tampermonkey 文档](https://www.tampermonkey.net/documentation.php) | `@match` / `@require` / `@grant` / `@run-at` 的通用说明 |
| [ScriptCat 文档](https://learn.scriptcat.org/) | 中文脚本开发教程 |
| [Greasespot 教程](https://wiki.greasespot.net/Tutorials) | Greasemonkey 基础 |
| [Match patterns](https://developer.chrome.com/docs/extensions/develop/concepts/match-patterns?hl=zh-cn) | `@match` 的匹配规则 |

> 本爬虫的 `@run-at` 与上游略有差异：`document-start` / `document-end` / `document-idle` 均支持，省略时为 `document-idle`，详见[Userscript重要配置说明](./Userscript重要配置说明.md)。

## JavaScript 类库

| 类库 | 用途 |
| :--- | :--- |
| [jQuery](https://api.jquery.com/) | DOM 查找与遍历；页面已自带时不要重复 `@require` |
| [ajax-hook](https://github.com/wendux/ajax-hook) | 拦截 `XMLHttpRequest`，读取异步接口返回值（`@run-at document-start`） |
| [browser-extension-url-match](https://github.com/lionel-rowe/browser-extension-url-match) | 在脚本里复用 `@match` 风格的 URL 匹配规则，ajaxHook 模板用它做接口筛选 |
| [jsDelivr](https://www.jsdelivr.com/) / [cdnjs](https://cdnjs.cloudflare.com/) | `@require` 的 CDN；优先选与页面自带版本一致的库 |

## 调试工具

| 工具 | 用途 |
| :--- | :--- |
| Chrome DevTools Console | 在真实页面上验证选择器与提取逻辑 |
| `chrome://inspect/#devices` | `remoteDebug: true` 时远程调试 WebView（需要设备开启 USB 调试） |
| `adb logcat -s GMSpider` | 查看 Java 层日志；`logLevel: "debug"` 时含 `MATCH:` / `BLOCK:` / `ALLOW:` / `CF_ALLOW:` |
| `adb logcat -s chromium` | 排查 WebView 崩溃（广告脚本、原生库缺失等） |
