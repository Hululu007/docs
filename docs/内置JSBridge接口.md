# 内置 JSBridge 接口

本爬虫支持大部分的 Greasemonkey API（GM_getValue、GM_setValue、GM_xmlhttpRequest 等），详情参考 [Android-WebMonkey](https://github.com/warren-bank/Android-WebMonkey)。

此外提供两套自定义 JSBridge：

## GmSpiderInject（爬虫专用）

暴露在 `window.GmSpiderInject`，用于爬虫与用户脚本之间的数据交互。

### GetSpiderArgs —— 获取当前执行的方法及参数

```javascript
GmSpiderInject.GetSpiderArgs()
```

返回 `JSON` 字符串，解析后为数组，第一项为方法名，余下依次为参数。

```javascript
const GMSpiderArgs = {};
if (typeof GmSpiderInject !== 'undefined') {
    let args = JSON.parse(GmSpiderInject.GetSpiderArgs());
    GMSpiderArgs.fName = args.shift();
    GMSpiderArgs.fArgs = args;
} else {
    // Chrome 浏览器调试时手动设置
    GMSpiderArgs.fName = "homeContent";
    GMSpiderArgs.fArgs = [true];
}
Object.freeze(GMSpiderArgs);
```

### SetSpiderResult —— 返回爬虫运行结果

```javascript
GmSpiderInject.SetSpiderResult(result)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `result` | `string` | JSON 编码字符串，内容格式取决于调用的爬虫方法（homeContent / detailContent 等） |

```javascript
if (typeof GmSpiderInject !== 'undefined') {
    GmSpiderInject.SetSpiderResult(JSON.stringify(result));
}
```

### ShowWebview / HideWebview —— 控制 WebView 可见性

```javascript
GmSpiderInject.ShowWebview()   // 显示 WebView
GmSpiderInject.HideWebview()   // 隐藏 WebView
```

显示行为与 `remoteDebug` 无关：`remoteDebug` 只开启 Chrome 远程调试并延迟销毁 WebView，**不会**让 WebView 可见。

> Cloudflare 挑战流程会自动显示 WebView，无需脚本调用 `ShowWebview()`。
> **不要在挑战文档里执行 `evaluateJavascript`**（包括用脚本去探测/轮询 DOM），否则会打断 CF 采集信号。

### 挑战期间的 SetSpiderResult

处于**已确认**的 Cloudflare 挑战期间时，`SetSpiderResult` 的结果会被丢弃（本次调用最终按超时结束）。应当在用户通过验证、页面回到正式文档后再返回结果；通行 Cookie 会保留给下一次调用。

## WebViewWM（Greasemonkey 桥接扩展）

暴露在 `window.WebViewWM`，提供浏览器功能 API 给用户脚本调用：

| 方法 | 说明 |
|------|------|
| `WebViewWM.getUrl(...)` | 获取当前 WebView URL |
| `WebViewWM.resolveUrl(..., url, base)` | 拼接相对 URL |
| `WebViewWM.loadUrl(..., url, headers)` | WebView 加载新 URL |
| `WebViewWM.getUserAgent(...)` | 获取当前 User-Agent |
| `WebViewWM.setUserAgent(..., ua)` | 设置 User-Agent |
| `WebViewWM.removeAllCookies(...)` | 清除所有 Cookie |
| `WebViewWM.toast(..., duration, text)` | 弹出 Toast 提示 |
| `WebViewWM.exit(...)` | 关闭当前页面 |
| `WebViewWM.download(..., uuid, mime, name)` | 下载文件 |
