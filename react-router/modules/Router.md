# Router.js

## 源码以及解析

```js
import React from "react";
import PropTypes from "prop-types";
import warning from "tiny-warning";

import HistoryContext from "./HistoryContext.js";
import RouterContext from "./RouterContext.js";

/**
 * The public API for putting history on context.
 */
class Router extends React.Component {
  // 该方法用于生成根路径的 match 对象
  static computeRootMatch(pathname) {
    return { path: "/", url: "/", params: {}, isExact: pathname === "/" };
  }

  constructor(props) {
    super(props);

    // 从传入的 history 实例中取了 location 对象存到了state里, 后面会通过setState更改location来触发重新渲染
    // location 对象包含 hash/pathname/search/state 等属性, 其实就是当前的路由信息
    this.state = {
      location: props.history.location,
    };

    // 简单来说, 由于子组件会比父组件更早完成挂载, 如果在 componentDidMount 进行监听, 则有可能在监听事件注册之前 history.location 已经发生改变, 因此我们需要在 constructor 中就注册监听事件, 并把改变的 location 记录下来, 等到组件挂载完了以后, 再更新到 state 上去
    // 其实如果去掉这部分的hack, 这里只是简单地设置了路由监听, 并在路由改变的时候更新 state 中的路由信息

    // This is a bit of a hack. We have to start listening for location
    // changes here in the constructor in case there are any <Redirect>s
    // on the initial render. If there are, they will replace/push when
    // they mount and since cDM fires in children before parents, we may
    // get a new location before the <Router> is mounted.

    // 判断组件是否已经挂载, componentDidMount阶段会赋值为true
    this._isMounted = false;
    // 储存在构造函数执行阶段发生改变的location
    this._pendingLocation = null;

    // 判断是否处于服务端渲染 (staticContext 是 staticRouter 传入<Router>的属性, 用于服务端渲染)
    if (!props.staticContext) {
      // 使用 history.listen() 添加路由监听事件
      this.unlisten = props.history.listen((location) => {
        if (this._isMounted) {
          // 如果组件已经挂载, 直接更新 state 的 location
          this.setState({ location });
        } else {
          // 如果组件未挂载, 就先把 location 存起来, 等到 didMount 阶段再 setState 
          this._pendingLocation = location;
        }
      });
    }
  }

  // 对应构造函数里的处理
  componentDidMount() {
    // 标记组件已挂载
    this._isMounted = true;

    if (this._pendingLocation) {
      // 将组件未挂载阶段的 location 变化 setState 上去
      this.setState({ location: this._pendingLocation });
    }
  }

  // 组件卸载前 解绑路由监听
  componentWillUnmount() {
    if (this.unlisten) {
      this.unlisten();
      this._isMounted = false;
      this._pendingLocation = null;
    }
  }

  render() {
    return (
      <RouterContext.Provider
        value={{
          history: this.props.history,
          // 当前的路由信息
          location: this.state.location,
          // 是否为跟路由
          match: Router.computeRootMatch(this.state.location.pathname),
          // staticContext: 用于服务端渲染
          staticContext: this.props.staticContext,
        }}
      >
        // 猜测一下 这是给 useHistory 用的
        <HistoryContext.Provider
          // 透传子组件
          children={this.props.children || null}
          value={this.props.history}
        />
      </RouterContext.Provider>
    );
  }
}

if (__DEV__) {
  Router.propTypes = {
    children: PropTypes.node,
    history: PropTypes.object.isRequired,
    staticContext: PropTypes.object,
  };

  Router.prototype.componentDidUpdate = function (prevProps) {
    warning(
      prevProps.history === this.props.history,
      "You cannot change <Router history>"
    );
  };
}

export default Router;
```

## 记录

1. 只有 DEV 环境才会有类型定义，以及警告信息。 只用写法感觉不错哎，侵入性不高。
