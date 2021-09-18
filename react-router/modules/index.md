# index.js

## 源码

```js
if (__DEV__) {
  if (typeof window !== "undefined") {
    const global = window;
    const key = "__react_router_build__";
    const buildNames = { cjs: "CommonJS", esm: "ES modules", umd: "UMD" };

    if (global[key] && global[key] !== process.env.BUILD_FORMAT) {
      const initialBuildName = buildNames[global[key]];
      const secondaryBuildName = buildNames[process.env.BUILD_FORMAT];

      // TODO: Add link to article that explains in detail how to avoid
      // loading 2 different builds.
      throw new Error(
        `You are loading the ${secondaryBuildName} build of React Router ` +
          `on a page that is already running the ${initialBuildName} ` +
          `build, so things won't work right.`
      );
    }

    global[key] = process.env.BUILD_FORMAT;
  }
}

// A <Router> that keeps the history of your “URL” in memory (does not read or write to the address bar). Useful in tests and non-browser environments like React Native.
export { default as MemoryRouter } from "./MemoryRouter.js";
// Used to prompt the user before navigating away from a page. When your application enters a state that should prevent the user from navigating away (like a form is half-filled out), render a <Prompt>.
export { default as Prompt } from "./Prompt.js";
// Rendering a <Redirect> will navigate to a new location. The new location will override the current location in the history stack, like server-side redirects (HTTP 3xx) do
export { default as Redirect } from "./Redirect.js";
// The Route component is perhaps the most important component in React Router to understand and learn to use well. Its most basic responsibility is to render some UI when its path matches the current URL.
export { default as Route } from "./Route.js";
// The common low-level interface for all router components. Typically apps will use one of the high-level routers instead:
//      <BrowserRouter>
//      <HashRouter>
//      <MemoryRouter>
//      <NativeRouter>
//      <StaticRouter>
// The most common use-case for using the low-level <Router> is to synchronize a custom history with a state management lib like Redux or Mobx. Note that this is not required to use state management libs alongside React Router, it’s only for deep integration.
export { default as Router } from "./Router.js";
// A <Router> that never changes location.
// This can be useful in server-side rendering scenarios when the user isn’t actually clicking around, so the location never actually changes. Hence, the name: static. It’s also useful in simple tests when you just need to plug in a location and make assertions on the render output
export { default as StaticRouter } from "./StaticRouter.js";
// Renders the first child <Route> or <Redirect> that matches the location.
export { default as Switch } from "./Switch.js";


// The generatePath function can be used to generate URLs to the routes. Internally the path-to-regexp library is used.
export { default as generatePath } from "./generatePath.js";
export { default as matchPath } from "./matchPath.js";
// You can get access to the history object’s properties and the closest <Route>'s match via the withRouter higher-order component. withRouter will pass updated match, location, and history props to the wrapped component whenever it renders
export { default as withRouter } from "./withRouter.js";


// 以下的不说了
export { default as __HistoryContext } from "./HistoryContext.js";
export { default as __RouterContext } from "./RouterContext.js";

export { useHistory, useLocation, useParams, useRouteMatch } from "./hooks.js";
```

## 解析

### 1. `__DEV__` 是什么 ？

通过 [babel-plugin-dev-expression](https://www.npmjs.com/package/babel-plugin-dev-expression) 插件注入的。
解析时将 `__DEV__` 替换成 process.env.NODE_ENV !== 'production'

### 2. initialBuildName 与 secondaryBuildName

```js
const initialBuildName = buildNames[global[key]];
const secondaryBuildName = buildNames[process.env.BUILD_FORMAT];

global[key] = process.env.BUILD_FORMAT

// global[key] 是在最后赋值的
```

### 3. export 与 import 的复合写法

```js
export { default as MemoryRouter } from "./MemoryRouter.js";
```

等同于（注意 不是等价！！！）

```js
import MemoryRouter from "./MemoryRouter.js";
// 这边编辑器是报错的 所以实际代码是不能这样写的, 也不可能用 export { MemoryRouter } 来代替, 意思就不一样了
export MemoryRouter;
```

```js
export { default as MemoryRouter } from "./MemoryRouter.js";
```

export 和 import 语句可以结合在一起，写成一行。
但需要注意的是，写成一行以后，MemoryRouter 实际上并没有被导入当前模块，只是相当于对外转发了这个接口，导致当前模块不能直接使用 MemoryRouter。

### 4. es 文件夹 和 modules 文件夹的关系?

在我目前看来 es 文件夹是依赖于打包后的 esm 文件夹, 而 esm 文件夹来源于 rollup 打包 modules 文件夹后得来的。
