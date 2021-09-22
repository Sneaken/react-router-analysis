# package.json

## peerDependencies 和 dependencies

```json
{
  "peerDependencies": {
    "react": ">=15"
  },
  "dependencies": {
    "@babel/runtime": "^7.12.13",
    "history": "^4.9.0",
    "loose-envify": "^1.3.1",
    "prop-types": "^15.6.2",
    "react-router": "5.2.1",
    "tiny-invariant": "^1.0.2",
    "tiny-warning": "^1.0.0"
  }
}
```

因为 react-router 是 dependencies 依赖, 所以安装 react-router-dom 的时候不需要在项目里显式安装。

直接 yarn add react-router-dom 或者 npm install react-router-dom 即可
