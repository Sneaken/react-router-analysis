# generatePath.js

## 源码与分析

```js
import pathToRegexp from "path-to-regexp";

const cache = {};
const cacheLimit = 10000;
let cacheCount = 0;

function compilePath(path) {
  // 如果存在缓存, 则直接返回缓存的generator
  if (cache[path]) return cache[path];

  const generator = pathToRegexp.compile(path);

  if (cacheCount < cacheLimit) {
    cache[path] = generator;
    cacheCount++;
  }

  return generator;
}

/**
 * Public API for generating a URL pathname from a path and parameters.
 */
function generatePath(path = "/", params = {}) {
  // generatePath('/user/:id', {id: 8}) => '/user/8'
  // generatePath('/user/:id', {user: 8}) => 报错 说 id should be a string
  return path === "/" ? path : compilePath(path)(params, { pretty: true });
}

export default generatePath;
```
