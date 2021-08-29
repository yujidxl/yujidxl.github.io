---
title: 实现一个plugin
date: 2021-08-01
tags:
  - webpack
categories:
  - webpack
---

### 1. webpack中plugin的作用
::: tip
插件是 webpack 的 支柱 功能。webpack 自身也是构建于你在 webpack 配置中用到的相同的插件系统之上！
插件目的在于解决 loader 无法实现的其他事。
:::

### 2. 怎样实现一个plugin，我们从哪开始？
:::tip
webpack 插件是一个具有 apply 方法的 JavaScript 对象。apply 方法会被 webpack compiler 调用，并且在 整个 编译生命周期都可以访问 compiler对象。
:::
```js
const pluginName = 'ConsoleLogOnBuildWebpackPlugin';

class ConsoleLogOnBuildWebpackPlugin {
  apply(compiler) {
    compiler.hooks.run.tap(pluginName, (compilation) => {
      console.log('webpack 构建过程开始！');
    });
  }
}

module.exports = ConsoleLogOnBuildWebpackPlugin;
```
> compiler hook 的 tap 方法的第一个参数，应该是驼峰式命名的插件名称。建议为此使用一个常量，以便它可以在所有 hook 中重复使用。

### 3. 用法 
>由于插件可以携带参数/选项，你必须在 webpack 配置中，向 plugins 属性传入一个 new 实例。
取决于你的 webpack 用法，对应有多种使用插件的方式。
配置方式 
webpack.config.js
```js
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 访问内置的插件
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    filename: 'my-first-webpack.bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: 'babel-loader',
      },
    ],
  },
  plugins: [
    new webpack.ProgressPlugin(),
    new HtmlWebpackPlugin({ template: './src/index.html' }),
  ],
};
```

### 4. 了解webpack plugin中可以提供给我们那些钩子调用
+ compiler 钩子
+ compilation 钩子
+ ContextModuleFactory Hooks
+ JavascriptParser Hooks
+ NormalModuleFactory Hooks 
具体细节详见[官方文档:](https://webpack.docschina.org/api/compiler-hooks/#watching)

### 5. 说一下hooks根据同步还是异步来分类时的种类
+ syncHook
+ AsyncSeriesHook
+ AsyncParallelHook
##### 他们分别代表什么呢？
- Sync. A sync hook can only be tapped with synchronous functions (`using myHook.tap()`).

- AsyncSeries. An async-series hook can be tapped with synchronous, callback-based and promise-based functions (`using myHook.tap(), myHook.tapAsync() and myHook.tapPromise()`). They call each async method in a row.

- AsyncParallel. An async-parallel hook can also be tapped with synchronous, callback-based and promise-based functions (`using myHook.tap(), myHook.tapAsync() and myHook.tapPromise()`). However, they run each async method in parallel.

### 6. 说一下hooks在绑定多个回调函数时怎么运行来分类
- Basic hook (without “Waterfall”, “Bail” or “Loop” in its name). This hook simply calls every function it tapped in a row.

- Waterfall. A waterfall hook also calls each tapped function in a row. Unlike the basic hook, it passes a return value from each function to the next function.

- Bail. A bail hook allows exiting early. When any of the tapped function returns anything, the bail hook will stop executing the remaining ones.

- Loop. When a plugin in a loop hook returns a non-undefined value the hook will restart from the first plugin. It will loop until all plugins return undefined.

::: tip
以上对于钩子的注册形式，会根据当前钩子指定的名称来确定, 会根据注册在对应钩子上的函数调用，来实现webpack中plugin的事件调用流。
:::

### 7. 我们来结合几个例子来理解上述概念
1. 我们定义一个ApplySay的类，来验证compiler中的done这个hook的注册函数是否会在对应时间被调用
``` js
module.exports =  class ApplySay {
  apply(compiler) {
    compiler.hooks.done.tap('applySay', (stats) => {
      console.log('done, success! ');
    })
  }
}
```
webpack.config.js
``` js
const {resolve} = require('path');
const {CleanWebpackPlugin} = require('clean-webpack-plugin');
const ApplySay = require('./plugins/ApplySay');
// const FileListPlugin = require('./plugins/FileListPlugin');
module.exports = {
  mode: 'production',
  entry: './src/main.js',
  output: {
    path: resolve(__dirname, 'dist'),
    filename: '[name].[contenthash:8].js'
  },
  module: {
    rules: [{
      test: /\.m?js$/,
      exclude: /node_modules/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env'] 
        }
      },
    }]
  },
  plugins: [new CleanWebpackPlugin(), new ApplySay(), 
    // new FileListPlugin(
    // {filename: 'new-file-list.md'}
    // )
  ]
}
```

2. 实现一个记录输出文件的md文件
FileListPlugin.js
``` js
module.exports = class FileListPlugin {
	constructor(options) {
		this.filename =
			options && options.filename ? options.filename : "file-list.md";
	}
	apply(compiler) {
		compiler.hooks.emit.tapAsync("FileListPlugin", (compilation, cb) => {
			// 通过 compilation.assets 获取文件数量
			const keys = Object.keys(compilation.assets),
				len = keys.length,
				// 添加统计信息
				content = Object.keys(compilation.assets).reduce(
					(acc, filename) => `${acc}- ${filename}\n`,
					`# ${len} file${len > 1 ? "s" : ""} emitted by webpack\n\n`,
				);

			// 往 compilation.assets 中添加清单文件
			compilation.assets[this.filename] = {
				// 写入新文件的内容
				source: function () {
					return content;
				},
				// 新文件大小（给 webapck 输出展示用）
				size: function () {
					return content.length;
				},
			};

			// 执行回调，让 webpack 继续执行
			cb();
		});
	}
};
```
webapck.config.js
``` js
const {resolve} = require('path');
const {CleanWebpackPlugin} = require('clean-webpack-plugin');
const ApplySay = require('./plugins/ApplySay');
+ const FileListPlugin = require('./plugins/FileListPlugin');
module.exports = {
  mode: 'production',
  entry: './src/main.js',
  output: {
    path: resolve(__dirname, 'dist'),
    filename: '[name].[contenthash:8].js'
  },
  module: {
    rules: [{
      test: /\.m?js$/,
      exclude: /node_modules/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env'] 
        }
      },
    }]
  },
  plugins: [new CleanWebpackPlugin(), new ApplySay(), 
  +  new FileListPlugin(
  +  {filename: 'new-file-list.md'}
  +  )
  ]
}
```
::: tip
 #### `总结`：
 1. plugin是定义一个类class
 2. plugin类中定义一个apply的方法，改方法返回compiler钩子
 3. 利用compiler钩子中程序执行的不用时间的钩子，插入我们需要的处理函数，实现plugin
 4. 我们在注册事件时需要区分当前hook的调用方式，明确是用tap，tapAsync或tapPromise
:::
