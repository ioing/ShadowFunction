---
description: 一个字符串执行的 worker 方法。
---

# WorkerFunction

```javascript
let workerFn = new WorkerFunction('postMessage('You said: ' + e.data)')
workerFn.onmessage = function (event) {
  console.log('Received message ' + event.data);
  doSomething();
}
workerFn.postMessage('Work done!');
```

