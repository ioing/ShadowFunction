---
description: 该方法会对沙盒中创建的虚拟节点是一个“平行宇宙节点”（平行 DOM），沙盒中的所有操作和事件将被安全同步到外部。
---

# ShadowDocument

## API：

| Api | Arguments | Retun |
| :--- | :--- | :--- |
| ShadowDocument | root\[Element\], template\[string\], setting\[object\] | ShadowFunction |

new 一个 ShadowDocument 时可传入三个参数，分别是插入到外部的节点以及要被执行的模版和鉴权设置，执行后返回一个 ShadowFunction 实例。

## 例子

每一次 new 时都会重新分配一个新的沙盒，需要注意尽量重复使用。

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

