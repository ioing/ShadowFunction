---
description: 该篇列举对象在 ShadowFunction 中的使用都将受到限制，以及限制级使用中的注意事项。
---

# Example

## alert\(\)

使用该函数时能够正确运行，但不会有任何响应。

```javascript
new ShadowFunction('alert(520)')()  // 正确执行但不会有任何响应
```

如果需要提供给内部代码使用 alert 方法时，可以如下：

```javascript
new ShadowFunction('alert(520)')({
    alert: function (str) {
        // 可以在这里做一层弹窗内容的可用性校验。
        filter(str).then(success => {
            alert(str)
        })  
        // 也可以在这里发送统计，以预防安全危机。
        http.post({
            data: {msg: str}
        })
    }
})
```

## console.log\(\)

使用该函数时能够正确运行，但不会有任何响应。

```javascript
new ShadowFunction('console.log(520)')()  // 正确执行但不会有任何响应
```

和 alert 相同，如果需要使用该方法时同样需要代入入参使用：

```javascript
new ShadowFunction('console.log(520)')({console})
```

开放给内部使用 console 函数时一般是给 ISV 调试使用的，正是情况下应禁止传入

## document

可以调用 document 的方法，但返回空和无响应。

```javascript
new ShadowFunction('console.log(document.cookie)')({console})  // 返回空 ''
```

下面是一个错误的使用例子，该错误用例不会造成风险，但是不建议直接传入 document 对象。

```javascript
new ShadowFunction('console.log(document.cookie)')({console, document}) // 不会泄漏 cookie，除非设置 cookie 为白名单
```

使用 document 中的方法时

```javascript
new ShadowFunction('document.createElement("div")')() // 可以正常创建节点
```

虽然 document 方法能够正确的创建节点，但是代码段创建的节点无法干扰到外部，除非将创建好的内部节点传入给外部，且外部代码把该节点插入到外部文档中。⚠️注意：这是非常危险的操作，因为一旦将内部节点注入到外部，内部代码就掌控了操作权。

那如何正确的让内部代码操作外部节点呢？

```javascript
new ShadowFunction('document.createElement("div")')({
    document: {
        // 提供可信的创建节点方法
        creatElement: function (tag) {
            if (['div', 'p', 'span'].indexOf(tag)) {
                return new ShadowFunction('return fn()')({
                    fn: function () {
                        return document.createElement(tag)
                    }
                })
            }
            return null
        }
        // 提供指定容器的选择器方法
        getElementById: function (id) {
            filter(id)  // 可以在这里进行 id 鉴权
        }
    }
})
```

通过安全创建和限定作用域选择器可以限制内部代码对 Dom 的权限，但即便可操作元素被限定依然有不安全风险，因此应该使用 AccessPermission 来对 Element 类方法进行鉴权操作：

```javascript
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

shadowFunction(`
  document.appendChild(document.createElement("div"))
`)({document})
```

## window

内部代码在访问 window、parent、top 等属性时均返回为空。

```javascript
new ShadowFunction('console.log(window)')({console}) // null
```

禁止直接将 window 作为入参传入，如下就是一个错误的使用例子：

```javascript
new ShadowFunction('console.log(window)')({console, window}) // ❌错误的例子
```

## prototype

内部代码不允许访问属性 prototype、\_\_proto\_\_、constructor 等属性。

```javascript
new ShadowFunction('console.log(a.prototype)')({console, a: {}}) // undefined
```

## 行为事件

当内部代码访问这些属性时可以通过统计行为来测绘 ISV 代码的安全地图，为日后安全分析做数据支撑。

```javascript
let tracker = (e) => { ... // 统计函数 }
new ShadowFunction('console.log(a.prototype = 110)')({console, a: {}}, e => {
    console.log(e) // {target: a, name: prototype, action: 'write', value: 110}
    tracker(e) // 行为对象、name、行为[read、write]、是否是原生属性、写入值
})
```

## 函数 return

传入函数时，如果函数有回调时需要在新的 ShadowFunction 中执行。如下例子中传入对象 setInterval 在执行回调时是在沙箱中执行回调函数的，而不是直接执行

```javascript
let shadowDoc
shadowDoc = new ShadowDocument(document.body, `
    <div>123</div>
    <div>
        <div>345</div>
    </div>
`)
shadowDoc(`
    document.body.append($template.content);
    console.log(123, document.body.getElementsByTagName("div")); 
    let i = 0; 
    setInterval(function () {
        document.body.getElementsByTagName("div")[0].setAttribute("a", i);
        document.body.getElementsByTagName("div")[1].innerText = i;
         i++; 
    }, 5000);
    document.body.getElementsByTagName("div")[0].addEventListener("click", function (e) {
        document.body.getElementsByTagName("div")[0].innerText = Date.now(); 
        console.log(e.pageX)
    }, false)
`)({
    console, 
    setInterval: function (fn, t) {
        setInterval(function () {
            new ShadowFunction('fn()')({fn})
        }, t)
    }
})
```

每次 new 都会创建一个新的实例，同段代码建议如下：

```javascript
let shadowDoc
shadowDoc = new ShadowDocument(document.body, `
    <div>...</div>
`)
shadowDoc(`
    ...
`)({
    console, 
    setInterval: function (fn, t) {
        setInterval(function () {
            shadowDoc('fn()')({fn})
        }, t)
    }
})
```



