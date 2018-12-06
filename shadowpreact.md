---
description: 基于 ShadowDocument 封装的 Preact 库沙盒，该沙盒中运行默认即可使用 Preact
---

# ShadowPreact

使用 Preact 创建一个简单的页面。

```javascript
new ShadowPreact(`
  document.body.append($template.content);
  preact.render(preact.createElement(
    "div",
    { id: "foo" },
    preact.createElement(
      "img",
      {src: "https://reactchina.sxlcdn.com/uploads/default/original/2X/4/4ac940018c8bbb978b015da531b8007221333e94.png"}
    ),
    preact.createElement(
      "button",
      { 
        onClick: function onClick(e) {
          alert("hi!");
        } 
      },
      "Click Me"
    )
  ),  document.getElementById('app'));
  console.log(document.getElementById('app'))
`)({console, alert: function (str) {alert(str)}, setInterval: function (fn, t) {console.log(t);setInterval(function () {fn();}, t)}})
```

ShadowPreact 返回一个 ShadowFunction 实例。

