# Userscript重要配置说明

关于Userscript的编写可以参考

https://learn.scriptcat.org/

https://www.tampermonkey.net/documentation.php

https://wiki.greasespot.net/Tutorials

本文主要讲解重点需要关注的几个配置项

## @run-at

注入脚本的时机。本爬虫支持：

| 值 | 行为 |
|----|------|
| `document-start` | 立即注入（此时 `document` 可能还没有 body） |
| `document-end` | 包在 `DOMContentLoaded` 回调里执行；`document-body` 也按此处理 |
| `document-idle` | 包在 `window load` 回调里执行；**省略 `@run-at` 时的默认值** |

当前构建开启了 `emulateOnPageFinished`，因此三种时机都在 `onPageStarted` 时注入脚本、只是执行点不同：

- `document-start`：注入即执行，能跑赢站点自己的脚本；
- `document-end`：等 `DOMContentLoaded`；
- `document-idle`：等 `window load`。重 JS / Rocket Loader 页面可能很晚或永不触发。

**需要 hook ajax（`XMLHttpRequest` / `fetch`）时用 `document-start`**，必须在站点的请求发出前替换掉原生方法。用 `document-end` 往往已经太晚，会漏掉首屏请求。

## @grant

本运行时在脚本闭包里**始终提供 `unsafeWindow`**（页面真实的 `window`），无需声明即可使用。模板里写 `@grant unsafeWindow` 只是保持与 Tampermonkey 的兼容，并不是启用它的开关。

需要关闭 JS 沙箱、让脚本直接跑在页面上下文时：

```javascript
// @grant none
```

其他 `@grant GM_*` 用于声明脚本会用到的 GM API。

## @match

脚本匹配的网址，确保爬虫所有牵扯到的域名都备规则包含

具体的匹配规则编写可以参考 [Match patterns](https://developer.chrome.com/docs/extensions/develop/concepts/match-patterns?hl=zh-cn)

## @require 

引用外部脚本，可以引用一些封装好的的js库