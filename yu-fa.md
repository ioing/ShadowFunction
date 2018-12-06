---
description: 将传入字符以 JavaScript 运行，并能限制内部运行时的各种安全问题，且能安全的将外部函数和对象传递给内部执行。
---

# ShadowFunction

| Api | Arguments | Retuen |
| :--- | :--- | :--- |
| ShadowFunction | setting\[object\] | 实例 \[shadowFunction\] |
| ShadowFunction | script\[string\] | 函数实例\[Function\] |
| 实例 \[shadowFunction\] | 注入对象\[object\], tracker\[function\] | 实例 \[shadowFunction\] |
| \[shadowFunction\].run | script\[string\] | 函数实例\[Function\] |

## API：

执行 ShadowFunction \('script'\) 后返回一个沙盒函数实例，该实例接收一个 Object 入参，该入参将被注入到实例执行代码中调用。

一般使用例子:

在 Object 入参后面还接收一个 tracker 函数，用来统计沙盒代码的非法读写行为。

## 例子：

```javascript
import { ShadowFunction} from 'shadow-function'

new ShadowFunction('console.log(a + b)')({a: 1, b: 2, console})  // 3
```

鉴权设置使用例子：

```javascript
import { ShadowFunction } from 'shadow-function'
let shadowFunction

shadowFunction = new ShadowFunction({
    Node: [
      'nodeName',
      'nodeType',
      'textContent'
    ],
    Element: [
      'style',
      'onblur',
      'onfocus',
      'onscroll',
      'offsetWidth',
      'offsetHeight',
      'clientWidth',
      'clientHeight',
      'innerText',
      'setAttribute',
      'removeAttribute',
      'createTextNode',
      'addEventListener',
      'getElementsByTagName'
    ],
    HTMLDivElement: []
})

shadowFunction = shadowFunction(`
  let y = 9 ; 
  trigger(function (){ 
    console.log(arguments.callee.caller, y)
  })`)
  
shadowFunction({
    console,
    trigger: (fn) => {
    	new ShadowFunction('fn()')({fn})
	}
}, e => {
  console.log(e) // 行为统计
})

```

设置 Element 类的鉴权时，如果设置了 DIV 标签的权限白名单，HTMLDivElement 将会依次合并原型链类型，如继承 Element、Node 白名单属性。

