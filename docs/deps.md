# 依赖分析，查找无用文件

对小程序的页面和组件进行依赖分析，找出未被引用的文件，生成 `packOptions` 项，在开发者工具上传代码时忽略无用文件。

支持小程序/插件，仅对 `wxml`、`wxss`、`wxs`、`js`、`json` 以及组件进行分析，不包括组件内的图片等资源。

需要注意的是，`js` 文件的依赖，支持 `import` 和 `require` 导入的模块，但运行时计算的路径如 `require(a + b)` 将无法识别。

## 用法

```js
Usage: miniprogram-slim analyzer [options]

Analyze dependencies of miniprogram, find out unused files

Options:
  -o, --output [dir]   path to directory for result (default: "./analyzer")
  -i, --ignore <glob>  glob pattern for files what should be excluded from unused files
  -w, --write          overwrite old project.config.json
  -t, --table          print miniprogram file size data
  -h, --help           output usage information
```

进入包含 `project.config.json` 的项目根目录，执行 `miniprogram-slim analyzer`，默认会生成 `./analyzer/result.json` 文件，记录生成的数据结果。

```json
{
  "packOptions": {
    "ignore": []
  },
  "dependencies": {
    "app": {
      "esDeps": [],
      "wxmlDeps": [],
      "wxssDeps": [],
      "compDeps": [],
      "wxsDeps": [],
      "jsonDeps": [],
      "files": []
    },
    "pages": {},
    "subpackages": {}
  },
  "unusedFiles": [],
  "data": {}
}
```

* `packOptions` 字段记录着在开发者工具打包上传时可以被忽略的文件，拷贝该部分至 `project.config.json` 即可，执行 `miniprogram-slim analyzer -w` 将自动进行同步。

* `dependencies` 字段记录着文件间的依赖关系，按页面维护分割，包括与页面相关的 `wxml`、`wxss`、`js`、`wxs` 以及组件的引用。

* `unusedFiles` 为未引用的文件数组。

* `data` 为保持依赖关系的文件大小的集合，`test/minicode` 项目测试部分结果如下，其中后缀为 `.json` 的表示一个组件。


```
┌─────────────────────────┬─────────────────────────────────────────────────────────────────┬────────────────┬─────────┬────────────────┐
│ page                    │ file & comp                                                     │ stat size (kB) │ percent │ totalSize (kB) │
├─────────────────────────┼─────────────────────────────────────────────────────────────────┼────────────────┼─────────┼────────────────┤
│                         │ miniprogram_npm/weui-miniprogram/weui-wxss/dist/style/weui.wxss │ 46.54          │ 62.89%  │                │
│                         ├─────────────────────────────────────────────────────────────────┼────────────────┼─────────┤                │
│                         │ app.json                                                        │ 0.58           │ 0.78%   │                │
│                         ├─────────────────────────────────────────────────────────────────┼────────────────┼─────────┤                │
│                         │ app.js                                                          │ 0.08           │ 0.11%   │                │
│                         ├─────────────────────────────────────────────────────────────────┼────────────────┼─────────┤                │
│                         │ app.wxss                                                        │ 0.08           │ 0.11%   │                │
│ app                     ├─────────────────────────────────────────────────────────────────┼────────────────┼─────────┤ 74             │
│                         │ util/util-c.js                                                  │ 0.04           │ 0.05%   │                │
│                         ├─────────────────────────────────────────────────────────────────┼────────────────┼─────────┤                │
│                         │ miniprogram_npm/weui-miniprogram/cell/cell.json                 │ 11.35          │ 15.34%  │                │
│                         ├─────────────────────────────────────────────────────────────────┼────────────────┼─────────┤                │
│                         │ miniprogram_npm/weui-miniprogram/searchbar/searchbar.json       │ 9.13           │ 12.34%  │                │
│                         ├─────────────────────────────────────────────────────────────────┼────────────────┼─────────┤                │
│                         │ miniprogram_npm/weui-miniprogram/cells/cells.json               │ 6.2            │ 8.38%   │                │
├─────────────────────────┼─────────────────────────────────────────────────────────────────┼────────────────┼─────────┼────────────────┤
│                         │ miniprogram_npm/miniprogram-sm-crypto/index.js                  │ 56.6           │ 55.85%  │                │
│                         ├─────────────────────────────────────────────────────────────────┼────────────────┼─────────┤                │
│                         │ miniprogram_npm/jsbn/index.js                                   │ 43.76          │ 43.18%  │                │
│                         ├─────────────────────────────────────────────────────────────────┼────────────────┼─────────┤                │
│                         │ pages/other/other.js                                            │ 0.89           │ 0.88%   │                │
│ pages/other/other       ├─────────────────────────────────────────────────────────────────┼────────────────┼─────────┤ 101.35         │
│                         │ pages/other/other.wxml                                          │ 0.05           │ 0.05%   │                │
│                         ├─────────────────────────────────────────────────────────────────┼────────────────┼─────────┤                │
│                         │ pages/other/other.json                                          │ 0.03           │ 0.03%   │                │
│                         ├─────────────────────────────────────────────────────────────────┼────────────────┼─────────┤                │
│                         │ pages/other/other.wxss                                          │ 0.02           │ 0.02%   │                │
├─────────────────────────┼─────────────────────────────────────────────────────────────────┼────────────────┼─────────┼────────────────┤
```
