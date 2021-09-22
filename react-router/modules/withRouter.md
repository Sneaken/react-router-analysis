# withRouter.md

## 源码

```js
import React from "react";
import PropTypes from "prop-types";
import hoistStatics from "hoist-non-react-statics";
import invariant from "tiny-invariant";

import RouterContext from "./RouterContext.js";

/**
 * A public higher-order component to access the imperative API
 */
function withRouter(Component) {
  // Component.displayName 组件在react-dev-utils 的展示名
  // Component.name 组件的function name
  const displayName = `withRouter(${Component.displayName || Component.name})`;
  const C = (props) => {
    const { wrappedComponentRef, ...remainingProps } = props;

    return (
      <RouterContext.Consumer>
        {(context) => {
          invariant(
            context,
            `You should not use <${displayName} /> outside a <Router>`
          );
          return (
            <Component
              // 组件用的参数
              {...remainingProps}
              // router 的相关属性
              {...context}
              // 挂载 ref
              ref={wrappedComponentRef}
            />
          );
        }}
      </RouterContext.Consumer>
    );
  };

  C.displayName = displayName;
  C.WrappedComponent = Component;

  if (__DEV__) {
    C.propTypes = {
      wrappedComponentRef: PropTypes.oneOfType([
        PropTypes.string,
        PropTypes.func,
        PropTypes.object,
      ]),
    };
  }
  // hoistStatics的三个参数 （目标组件，源组件，屏蔽的静态方法字段）
  return hoistStatics(C, Component);
}

export default withRouter;
```

## 记录

1. 为什么使用 hoist-non-react-statics ?

> Copies non-react specific statics from a child component to a parent component. Similar to Object.assign, but with React static keywords blacklisted from being overridden.

是为了解决 HOC 组件上面静态方法丢失的问题， 挂载 ref 时，应该传给 wrappedComponentRef
