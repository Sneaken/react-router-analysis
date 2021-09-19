# Switch.js

## 源码

```js
import React from "react";
import PropTypes from "prop-types";
import invariant from "tiny-invariant";
import warning from "tiny-warning";

import RouterContext from "./RouterContext.js";
import matchPath from "./matchPath.js";

/**
 * The public API for rendering the first <Route> that matches.
 */
class Switch extends React.Component {
  render() {
    return (
      <RouterContext.Consumer>
        {context => {
          // 也应该被Router组件所包裹
          invariant(context, "You should not use <Switch> outside a <Router>");

          const location = this.props.location || context.location;

          let element, match;

          
          // We use React.Children.forEach instead of React.Children.toArray().find()
          // here because toArray adds keys to all child elements and we do not want
          // to trigger an unmount/remount for two <Route>s that render the same
          // component at different URLs.
          React.Children.forEach(this.props.children, child => {
            // React.isValidElement 校验是否是合法的 jsx 元素
            // 这边注意是双等号 所以 match 可以是 undefined 或者 null
            if (match == null && React.isValidElement(child)) {
              // 每次遍历都给 element 赋上当前的child
              element = child;

              // child 上传入的 path 或者 from(来自 Redirect 组件) 作为 path
              const path = child.props.path || child.props.from;

              // 这边的意思也很明了，如果没有 path ，那么这个child组件必定被渲染
              match = path
                ? matchPath(location.pathname, { ...child.props, path })
                // Router.computeRootMatch 生成的对象，其实就是根路由的 match 对象
                : context.match;
            }
          });

          // 所以 Switch 只会渲染匹配到的那一个组件, 其他组件是不会渲染的。
          return match
            // React.cloneElement: 如果 element 不是一个组件, 而是一个 dom 元素的话， React 会抛出警告⚠️
            // Warning: React does not recognize the `computedMatch` prop on a DOM element. If you intentionally want it to appear in the DOM as a custom attribute, spell it as lowercase `computedmatch` instead. If you accidentally passed it from a parent component, remove it from the DOM element.
            ? React.cloneElement(element, { location, computedMatch: match })
            : null;
        }}
      </RouterContext.Consumer>
    );
  }
}

if (__DEV__) {
  Switch.propTypes = {
    children: PropTypes.node,
    location: PropTypes.object
  };

  Switch.prototype.componentDidUpdate = function(prevProps) {
    warning(
      !(this.props.location && !prevProps.location),
      '<Switch> elements should not change from uncontrolled to controlled (or vice versa). You initially used no "location" prop and then provided one on a subsequent render.'
    );

    warning(
      !(!this.props.location && prevProps.location),
      '<Switch> elements should not change from controlled to uncontrolled (or vice versa). You provided a "location" prop initially but omitted it on a subsequent render.'
    );
  };
}

export default Switch;
```
