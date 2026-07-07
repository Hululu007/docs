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

### ShowWebview / HideWebview —— 调试用

```javascript
GmSpiderInject.ShowWebview()   // 显示 WebView（debug 模式有效）
GmSpiderInject.HideWebview()   // 隐藏 WebView
```

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
