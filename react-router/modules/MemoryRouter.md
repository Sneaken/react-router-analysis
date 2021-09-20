# MemoryRouter.js

## 源码

```js
import React from "react";
import PropTypes from "prop-types";
import { createMemoryHistory as createHistory } from "history";
import warning from "tiny-warning";

import Router from "./Router.js";

/**
 * The public API for a <Router> that stores location in memory.
 * 服务端常用？
 */
class MemoryRouter extends React.Component {
  history = createHistory(this.props);

  render() {
    return <Router history={this.history} children={this.props.children} />;
  }
}

if (__DEV__) {
  MemoryRouter.propTypes = {
    initialEntries: PropTypes.array,
    initialIndex: PropTypes.number,
    getUserConfirmation: PropTypes.func,
    keyLength: PropTypes.number,
    children: PropTypes.node,
  };

  MemoryRouter.prototype.componentDidMount = function () {
    warning(
      !this.props.history,
      "<MemoryRouter> ignores the history prop. To use a custom history, " +
        "use `import { Router }` instead of `import { MemoryRouter as Router }`."
    );
  };
}

export default MemoryRouter;
```
