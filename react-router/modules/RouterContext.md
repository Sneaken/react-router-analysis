# RouterContext.js

## 源码

```js
import createNamedContext from "./createNamedContext";

const context = /*#__PURE__*/ createNamedContext("Router");
export default context;
```

## 解析

1. 模拟 react 16.8+ 的 useContext hooks

2. `/*#__PURE__*/` 是干嘛的？

terser 的标记位

其中terser认可的行内注释一般有3种：
1. `/*@__INLINE__*/` - forces a function to be inlined somewhere.
2. `/*@__NOINLINE__*/` - Makes sure the called function is not inlined into the call site.
3. `/*@__PURE__*/` - Marks a function call as pure. That means, it can safely be dropped.
可以把@换成#。

inline 表示内联，会强制把标记的函数里依赖的函数内敛在里面。
noinline 则相反，不会把函数拆成内容内联进函数体内。
pure 会告诉编辑器，如果没用到这玩意，可以放心删了。
