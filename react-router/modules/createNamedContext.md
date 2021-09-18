# createNamedContext.js

## 源码

```js
// TODO: Replace with React.createContext once we can assume React 16+
import createContext from "mini-create-react-context";

const createNamedContext = (name) => {
  const context = createContext();
  //  Context object accepts a displayName string property. React DevTools uses this string to determine what to display for the context.
  context.displayName = name;

  return context;
};

export default createNamedContext;
```

## 解析

### 1. why use mini-create-react-context ?

> (A smaller) Polyfill for the React context API

主要还是应为 package.json 里面 react 版本范围 是 >= 15, ~~使用者可能用不了 React context API~~, 由于React16和15的Context互不兼容。

#### 依赖包

```json
{
  "peerDependencies": {
    "prop-types": "^15.0.0",
    "react": "^0.14.0 || ^15.0.0 || ^16.0.0 || ^17.0.0"
  }
}
```

### 2. Context.displayName ?

> Context object accepts a displayName string property. React DevTools uses this string to determine what to display for the context.

For example, the following component will appear as MyDisplayName in the DevTools:

```js
const MyContext = React.createContext(/* some value */);
MyContext.displayName = 'MyDisplayName';

<MyContext.Provider> // "MyDisplayName.Provider" in DevTools
<MyContext.Consumer> // "MyDisplayName.Consumer" in DevTools

```
