# Lifecycle.js

## 源码

```js
import React from "react";

// 主要封装了组件的各个生命周期
class Lifecycle extends React.Component {
  // didMount 阶段（挂载）
  componentDidMount() {
    if (this.props.onMount) this.props.onMount.call(this, this);
  }

  // didUpdate 阶段（更新）
  componentDidUpdate(prevProps) {
    if (this.props.onUpdate) this.props.onUpdate.call(this, this, prevProps);
  }

  // willUnmount 阶段（卸载）
  componentWillUnmount() {
    if (this.props.onUnmount) this.props.onUnmount.call(this, this);
  }

  // 不渲染
  render() {
    return null;
  }
}

export default Lifecycle;
```
