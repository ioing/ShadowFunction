---
description: 一个防范异常的 Jsonp 方法。
---

# Jsonp

该方法能防范基本的 jsonp 异常，并能在失败或网络重新连接后主动尝试请求。

```javascript
import { jsonp } from 'shadow-function'
jsonp({
  url: "http://suggest.taobao.com/sug?code=utf-8&q=iphoneX"
}).then((data) => {
  console.log("jsonp:", data)
})
```

注意：该方案只能防范异常对主文档的影响，并不能限制 jsonp 中的恶意代码。

