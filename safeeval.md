---
description: 一个简单的执行代码函数
---

# SafeEval

返回沙盒中的对象或做简单运算。如下：

```javascript
import { safeEval } from 'shadow-function'

safeEval('(10 * 10 + 2)') // 102
safeEval('({})') // 沙盒对象
```

该方法和 ShadowFunction 不同的是该方法是完全的死亡沙箱，不支持传递外部对象，只能用来做一些简单的逻辑和运算。

