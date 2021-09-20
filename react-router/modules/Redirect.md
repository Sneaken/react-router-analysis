# Redirect.js

## 源码以及解析

```js
import React from "react";
import PropTypes from "prop-types";
import { createLocation, locationsAreEqual } from "history";
import invariant from "tiny-invariant";

import Lifecycle from "./Lifecycle.js";
import RouterContext from "./RouterContext.js";
import generatePath from "./generatePath.js";

/**
 * The public API for navigating programmatically with a component.
 */
function Redirect({ computedMatch, to, push = false }) {
  return (
    <RouterContext.Consumer>
      {(context) => {
        invariant(context, "You should not use <Redirect> outside a <Router>");

        const { history, staticContext } = context;

        // 看是往路由记录里面推数据，还是替换当前的路由记录，默认是替换
        const method = push ? history.push : history.replace;
        // computedMatch 是由 <Switch> 传下来的
        const location = createLocation(
          computedMatch
            ? typeof to === "string"
              ? generatePath(to, computedMatch.params)
              : {
                  // to 是一个对象
                  // for example
                  // {
                  //   pathname: "/login",
                  //   search: "?utm=your+face",
                  //   state: { referrer: currentLocation }
                  // }
                  // 如果to 没有 pathname, 则默认为根路由 '/'
                  ...to,
                  pathname: generatePath(to.pathname, computedMatch.params),
                }
            : // 不被 <Switch> 所包裹，则直接由 to 属性构造 location
              to
        );

        // When rendering in a static context,
        // set the new location immediately.
        if (staticContext) {
          // 服务端渲染是没有生命周期的， 所以判断是服务端渲染时，直接执行 method 方法
          method(location);
          return null;
        }

        // 所以 Redirect 组件其实是不渲染的，只是负责跳转。
        return (
          <Lifecycle
            onMount={() => {
              method(location);
            }}
            onUpdate={(self, prevProps) => {
              // 当 to 发生变化时, 根据 to 属性 重新构造一个 location
              const prevLocation = createLocation(prevProps.to);
              if (
                !locationsAreEqual(prevLocation, {
                  ...location,
                  key: prevLocation.key,
                })
              ) {
                // 当 location 发生变化时 执行 method
                method(location);
              }
            }}
            // 只有 to 发生变化才会引起 Lifecycle 组件重新渲染。
            to={to}
          />
        );
      }}
    </RouterContext.Consumer>
  );
}

if (__DEV__) {
  Redirect.propTypes = {
    push: PropTypes.bool,
    from: PropTypes.string,
    to: PropTypes.oneOfType([PropTypes.string, PropTypes.object]).isRequired,
  };
}

export default Redirect;
```
