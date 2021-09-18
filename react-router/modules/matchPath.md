# matchPath.js

## 源码

```js
import pathToRegexp from "path-to-regexp";

const cache = {};
const cacheLimit = 10000;
let cacheCount = 0;

function compilePath(path, options) {
  const cacheKey = `${options.end}${options.strict}${options.sensitive}`;
  const pathCache = cache[cacheKey] || (cache[cacheKey] = {});

  if (pathCache[path]) return pathCache[path];

  const keys = [];
  const regexp = pathToRegexp(path, keys, options);
  const result = { regexp, keys };

  if (cacheCount < cacheLimit) {
    pathCache[path] = result;
    cacheCount++;
  }

  return result;
}

/**
 * Public API for matching a URL pathname to a path.
 * 路由匹配
 */
function matchPath(pathname, options = {}) {
  // 目前还没看到 options 是 字符串的情况，更别说 options 是数组了
  // 懂了 这边说的是匹配到的路径列表， 不， 你不懂

  // 正常情况下, options 是 <Route> 的 props, 是个对象;
  // 这里判断, 是为了兼容 `react-router-redux`库中某个调用传入的 options 只有 path
  if (typeof options === "string" || Array.isArray(options)) {
    options = { path: options };
  }

  // 取出路由路径 path 和匹配参数 exact 等, 并赋初值
  const { path, exact = false, strict = false, sensitive = false } = options;

  //  统一 path 类型 (path可能是数组形式['/', '/user']或字符串形式"/user")
  const paths = [].concat(path);

  // 这边用 reduce 的原因， 我认为是这样写比较优雅，可能用 for 循环会看的比较明白一点， 为什么不用 forEach， forEach 打断循环写起来可不优雅
  return paths.reduce((matched, path) => {
    //  path 不存在 就 return null;
    if (!path && path !== "") return null;
    // matched 有了, 就返回匹配到的结果
    if (matched) return matched;

    // 根据路由路径 path 和匹配参数 exact 等参数拼出正则 regexp, keys 是路径参数(比如/user:id的id)
    const { regexp, keys } = compilePath(path, {
      end: exact,
      strict,
      sensitive,
    });
    // 这是正则上面的 exec 方法
    const match = regexp.exec(pathname);

    // 没有匹配到直接返回 null;
    if (!match) return null;

    //
    const [url, ...values] = match;
    // 是不是精确匹配
    const isExact = pathname === url;

    // 要求准确匹配却没有全等匹配到, 返回 nul
    if (exact && !isExact) return null;

    return {
      path, // the path used to match
      url: path === "/" && url === "" ? "/" : url, // the matched portion of the URL
      isExact, // whether or not we matched exactly
      params: keys.reduce((memo, key, index) => {
        //  /user/:id => /user/8
        //  { id: 8}
        //  就是这个意思
        memo[key.name] = values[index];
        return memo;
      }, {}),
    };
  }, null);
}

export default matchPath;
```
