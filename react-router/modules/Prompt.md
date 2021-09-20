# Prompt.js

## 源码

```js
import React from "react";
import PropTypes from "prop-types";
import invariant from "tiny-invariant";

import Lifecycle from "./Lifecycle.js";
import RouterContext from "./RouterContext.js";

/**
 * The public API for prompting the user before navigating away from a screen.
 */
function Prompt({ message, when = true }) {
  return (
    <RouterContext.Consumer>
      {(context) => {
        invariant(context, "You should not use <Prompt> outside a <Router>");

        // 不支持服务端渲染？
        if (!when || context.staticContext) return null;

        // 路由跳转警告
        const method = context.history.block;

        return (
          <Lifecycle
            onMount={(self) => {
              // 注册一个简单的提示消息，在用户离开当前页面之前，该消息将显示给用户。
              // self.release 实际上是 unblock 函数
              self.release = method(message);
            }}
            onUpdate={(self, prevProps) => {
              if (prevProps.message !== message) {
                // message 变化时 重新注册
                self.release();
                self.release = method(message);
              }
            }}
            onUnmount={(self) => {
              // 组件卸载后取消挂载的函数
              self.release();
            }}
            message={message}
          />
        );
      }}
    </RouterContext.Consumer>
  );
}

if (__DEV__) {
  // message 支持是一个字符串或者是一个函数，其他类型的东西我试了下 没反应
  // 当message为一个函数时，参数为要跳转的路由location对象与路由动作，如刷新页面为POP，普通跳转为PUSH
  // message={(location, aciton) => {}}
  const messageType = PropTypes.oneOfType([PropTypes.func, PropTypes.string]);

  Prompt.propTypes = {
    when: PropTypes.bool,
    // message 是必填的
    message: messageType.isRequired,
  };
}

export default Prompt;
```
