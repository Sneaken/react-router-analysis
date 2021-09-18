# Route.js

## 源码

```js
import React from "react";
import { isValidElementType } from "react-is";
import PropTypes from "prop-types";
import invariant from "tiny-invariant";
import warning from "tiny-warning";

import RouterContext from "./RouterContext.js";
import matchPath from "./matchPath.js";

// 判断是否没有子组件
function isEmptyChildren(children) {
  return React.Children.count(children) === 0;
}

// 检验 children 的合法性， 不合法给出提示
function evalChildrenDev(children, props, path) {
  const value = children(props);

  warning(
    value !== undefined,
    "You returned `undefined` from the `children` function of " +
      `<Route${path ? ` path="${path}"` : ""}>, but you ` +
      "should have returned a React element or `null`"
  );

  return value || null;
}

/**
 * The public API for matching a single path and rendering.
 */
class Route extends React.Component {
  render() {
    return (
      <RouterContext.Consumer>
        {(context) => {
          // context 应该包含 history | location | match | staticContext

          // sogo  原来是如此判断是否被 Router 所包裹
          invariant(context, "You should not use <Route> outside a <Router>");

          const location = this.props.location || context.location;

          // match 对象的 来源优先级（Switch -> props.path -> context）
          // computedMatch 是 Switch 组件传下来的， 优先级最高
          const match = this.props.computedMatch
            ? this.props.computedMatch // <Switch> already computed the match for us
            : // Route 组件上的 path 属性，优先级第二
            this.props.path
            ? matchPath(location.pathname, this.props)
            : // 最后是 context 上的 match 对象
              context.match;

          // 重新组装 props， 往下透传
          const props = { ...context, location, match };

          let { children, component, render } = this.props;

          // 这边兼容了 Preact 因为 Preact 会传一个空数组来表示没有children，统一在没children 传 null
          // Preact uses an empty array as children by
          // default, so use null if that's the case.
          if (Array.isArray(children) && isEmptyChildren(children)) {
            children = null;
          }

          // <Route>组件提供的三种渲染方式, 优先级 children > component > render
          return (
            <RouterContext.Provider value={props}>
              {props.match
                ? children
                  ? typeof children === "function"
                    ? __DEV__
                      ? evalChildrenDev(children, props, this.props.path)
                      : children(props)
                    : children
                  : component
                  ? React.createElement(component, props)
                  : render
                  ? render(props)
                  : null
                : typeof children === "function"
                ? __DEV__
                  ? evalChildrenDev(children, props, this.props.path)
                  : children(props)
                : null}
            </RouterContext.Provider>
          );
        }}
      </RouterContext.Consumer>
    );
  }
}

if (__DEV__) {
  Route.propTypes = {
    children: PropTypes.oneOfType([PropTypes.func, PropTypes.node]),
    component: (props, propName) => {
      if (props[propName] && !isValidElementType(props[propName])) {
        return new Error(
          `Invalid prop 'component' supplied to 'Route': the prop is not a valid React component`
        );
      }
    },
    exact: PropTypes.bool,
    location: PropTypes.object,
    path: PropTypes.oneOfType([
      PropTypes.string,
      PropTypes.arrayOf(PropTypes.string),
    ]),
    render: PropTypes.func,
    sensitive: PropTypes.bool,
    strict: PropTypes.bool,
  };

  Route.prototype.componentDidMount = function () {
    warning(
      !(
        this.props.children &&
        !isEmptyChildren(this.props.children) &&
        this.props.component
      ),
      "You should not use <Route component> and <Route children> in the same route; <Route component> will be ignored"
    );

    warning(
      !(
        this.props.children &&
        !isEmptyChildren(this.props.children) &&
        this.props.render
      ),
      "You should not use <Route render> and <Route children> in the same route; <Route render> will be ignored"
    );

    warning(
      !(this.props.component && this.props.render),
      "You should not use <Route component> and <Route render> in the same route; <Route render> will be ignored"
    );
  };

  Route.prototype.componentDidUpdate = function (prevProps) {
    warning(
      !(this.props.location && !prevProps.location),
      '<Route> elements should not change from uncontrolled to controlled (or vice versa). You initially used no "location" prop and then provided one on a subsequent render.'
    );

    warning(
      !(!this.props.location && prevProps.location),
      '<Route> elements should not change from controlled to uncontrolled (or vice versa). You provided a "location" prop initially but omitted it on a subsequent render.'
    );
  };
}

export default Route;
```

## 包

[](https://www.npmjs.com/package/tiny-invariant)
