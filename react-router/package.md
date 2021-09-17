# package.json

## files

The optional files field is an array of file patterns that describes the entries to be included when your package is installed as a dependency.

### 必然显示的内容

1. package.json
2. README
3. LICENSE / LICENCE
4. The file in the "main" field

### 必然不显示的内容

1. .git
2. CVS
3. .svn
4. .hg
5. .lock-wscript
6. .wafpickle-N
7. .\*.swp
8. .DS_Store
9. .\_\*
10. npm-debug.log
11. .npmrc
12. node_modules
13. config.gypi
14. \*.orig
15. package-lock.json (use npm-shrinkwrap.json if you wish it to be published)

### 分析

```json
{
  "files": [
    "MemoryRouter.js",
    "Prompt.js",
    "Redirect.js",
    "Route.js",
    "Router.js",
    "StaticRouter.js",
    "Switch.js",
    "cjs",
    "es",
    "esm",
    "index.js",
    "generatePath.js",
    "matchPath.js",
    "modules/*.js",
    "modules/utils/*.js",
    "withRouter.js",
    "warnAboutDeprecatedCJSRequire.js",
    "umd"
  ]
}
```

综上所述, 必然会显示的文件有 `"MemoryRouter.js", "Prompt.js", "Redirect.js", "Route.js", "Router.js", "StaticRouter.js", "Switch.js", "index.js", "generatePath.js", "matchPath.js", "modules/*.js", "withRouter.js", "modules/utils/*.js", "warnAboutDeprecatedCJSRequire.js", "package.json", "README"`。
必然会显示的文件夹 `"cjs", "es", "esm", "umd", "modules"`。

## main

> CommonJS包的入口文件

The main field is a module ID that is the primary entry point to your program. That is, if your package is named foo, and a user installs it, and then does require("foo"), then your main module's exports object will be returned.

This should be a module relative to the root of your package folder.

For most modules, it makes the most sense to have a main script and often not much else.

If main is not set it defaults to index.js in the packages root folder.

for example:

if main = index.js
require('foo') => 实际导入的是 foo/index.js

## module

> 非 NPM 官方标准字段, rollup 提出的概念。
> ES Module包的入口文件

## sideEffects

> 非 NPM 官方标准字段, 而是 webpack 为了更好实现 tree shaking 所提出的配置项(Webpack4+)。
> 
> 默认值为 true

因此 webpack 中的 tree-shaking 既需要 webpack 中进行配置，也需要在 Package.json 中进行配置。

sideEffects 是通知 webpack 该模块是可以安全的 tree-shaking, 无需关心其副作用。

webpack 处理 sideEffects 逻辑：

1. 如果 sideEffects 配置为 false，该模块内代码都没有副作用，只要没有被导入使用，均会被删除，webpack 也无需分析
2. 如果 sideEffects 明确了某些文件，Webpack 打包时，即使这些文件中的有副作用的那部分代码没被使用，也会保留。

注意，生产模式会删除无副作用且没有导入的代码，开发模式都会保留，只是会标注
