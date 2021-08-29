---
title: vite 学习路程
date: 2021-03-23
tags:
  - 脚手架
categories:
  - 脚手架
---
## 简介
1. 产生的原因，
>开发环境下现有的脚手架webpack编译过程过于耗时，旨在为开发节约开发时间
2. 利用了现有浏览器支持native ES modules的特性，即原生支持`import`，`export`。
3. 拥有更快的HMR
4. 使用`Rollup`打包生产环境的代码，输出高度优化的资源文件。
5. 既提供了开箱即用的简单的配置，又支持高度可配置。

## 浏览器支持性
1. 开发环境下的浏览器要求是支持native es module即可
2. 生产环境下我们默认编译的文件是只能运行在支持native es module的浏览器中。但是我们可以使用一个plugin`@vitejs/plugin-legacy`以提供老的浏览器支持。

## 怎样实现多浏览器都可以适用？
::: tip
由于vite默认情况下只支持一些已经拥有本地ESM的浏览器，这个插件能够使得vite支持那些不支持本地ESM的浏览器。基于此，此插件会实现以下几个目的：
:::
  1. Generate a corresponding legacy chunk for every chunk in the final bundle, transformed with @babel/preset-env and emitted as SystemJS modules (code splitting is still supported!).

  2. Generate a polyfill chunk including SystemJS runtime, and any necessary polyfills determined by specified browser targets and actual usage in the bundle.

  3. Inject `<script nomodule>` tags into generated HTML to conditionally load the polyfills and legacy bundle only in browsers without native ESM support.

  4. Inject the `import.meta.env.LEGACY` env variable, which will only be `true` in the legacy production build, and `false` in all other cases. (requires `vite@^2.0.0-beta.69`)
#### 使用方式
```javascript
// vite.config.js
import legacy from '@vitejs/plugin-legacy'

export default {
  plugins: [
    legacy({
      targets: ['defaults', 'not IE 11']
    })
  ]
}
// defaults: 意味着编译最近主流的浏览器支持
```
了解更多可配置项 [github](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy)
::: tip
以上告一段落
:::
## 支持多页面配置
``` code
├── package.json
├── vite.config.js
├── index.html
├── main.js
└── nested
    ├── index.html
    └── nested.js
```

添加配置项： 
``` javascript
// vite.config.js
const { resolve } = require('path')
module.exports = {
  build: {
    rollupOptions: {
      input: {
        main: resolve(__dirname, 'index.html'),
        nested: resolve(__dirname, 'nested/index.html')
      }
    }
  }
}
```

## 使用外部函数库注入模块的形式配置
``` javascript
// vite.config.js
const path = require('path')

module.exports = {
  build: {
    lib: {
      entry: path.resolve(__dirname, 'lib/main.js'),
      name: 'MyLib'
    },
    rollupOptions: {
      // make sure to externalize deps that shouldn't be bundled
      // into your library
      external: ['vue'],
      output: {
        // Provide global variables to use in the UMD build
        // for externalized deps
        globals: {
          vue: 'Vue'
        }
      }
    }
  }
}
```

## 项目文件中可以使用的全局变量
``` javascript
import.meta.env.MODE // 例如当前环境 development或production
import.meta.env.BASE_URL // config中base的配置项
import.meta.env.PROD // 当前app是否运行在production环境下
import.meta.env.DEV //是否运行在开发环境下
```
