---
title: ssr框架搭建记录
date: 2022-06-29
tags:
  - ssr
categories:
  - ssr
---

### 构建工具搭建过程中的注意事项

1. 保证router，store，app实例的唯一性。所以写法都是需要导出相关实例的创建方法，工厂模式。
2. css 等样式文件的处理。
```js
// 去除mini-css-extract-plugin 防止css注入到html中使用document.createElement方法导致报错
const isExtracting = config.plugins.has("extract-css");
if (isExtracting && isProd) {
  //   // Remove extract
  const langs = ["css", "postcss", "scss", "sass", "less", "stylus"];
  const types = ["vue-modules", "vue", "normal-modules", "normal"];
  for (const lang of langs) {
    for (const type of types) {
      const rule = config.module.rule(lang).oneOf(type);
      rule.uses.delete("extract-css-loader");
      // Critical CSS
      if (isServer) {
        rule
          .use("css-context")
          .loader(CssContextLoader)
          .before("css-loader");
      } else {
        rule
          .use("vue-style-loader")
          .loader(VueStyleLoader)
          .tap((options) => ({
            ...options,
            injectType: "singletonStyleTag",
            sourceMap: false,
            shadowMode: false,
          }))
          .before("css-loader");
      }
    }
  }
  config.plugins.delete("extract-css");
}
```

### 基本流程总结
+ 在vue需要获取服务端数据的组件中定义asyncData方法（此处为规范统一取名为asyncData，其实什么方法都可，只要不与vue定义的原生属性方法重名就行）。
+ 在nodejs的express框架实现的路由，或koa实现的路由变更时，记录参数，和路径名称。
+ 调用server端的方法，传入context，根据当前访问的url，在客户端与服务端统一路由的onReady事件的回调中，匹配当前需要渲染的路由url对应的组件。
+ 执行组件内定义的asyncData，此方法返回Promise，在所有接口数据请求完毕后，then函数中执行resolve(app)操作，将store中填充好的数据，赋值给context.state，交由初始化了window.__INITIAL_STATE__变量，为后续客户端切换路由参数时需要store数据做准备。
+ 客户端router.onReady事件回调中router.beforeResolve((to, from, next))记录路由变化，比较前后路由变化，引起的组件改变，找出这些差异组件，并执行组件的asyncData方法后，再跳转路由，并且激活dom，挂载dom。


### 怎样实现一个开发环境可热更新的ssr架构
:::tip
总结来说需要监听文件修改，实时编译获取最新的vue-ssr-server-bundle.json，并且在处理请求的时候随时获取最新的vue-ssr-client-manifest.json，拿最新的客户端json文件和服务端json文件生成，renderString或者renderStream，传给客户端访问。
:::
>具体实现：
``` js
const webpack = require('webpack');
const axios = require('axios');
const MemoryFS = require('memory-fs');
const fs = require('fs');
const path = require('path');
const Router = require('@koa/router');

// webpack配置文件
const webpackConfig = require('@vue/cli-service/webpack.config');
const { createBundleRenderer } = require('vue-server-renderer');

// 编译webpack配置文件
  const serverCompiler = webpack(webpackConfig);
  const mfs = new MemoryFS();
  serverCompiler.outputFileSystem = mfs;

// 监听文件修改，实时编译获取最新的vue-ssr-server-bundle.json
let bundle;
serverCompiler.watch({}, (err, stats) => {
  if (err) {
    throw err;
  }
  const _stats = stats.toJson();
  _stats.errors.forEach(error => console.log(error));
  _stats.warnings.forEach(warn => console.warn(warn));
  const bundlePath = path.join(webpackConfig.output.path, 'vue-ssr-server-bundle.json');
  bundle = JSON.parse(mfs.readFileSync(bundlePath, 'utf-8'));
});

// 处理请求
const handleRequest = async ctx => {
  console.log('path', ctx.path, ctx.query);
  if (!bundle) {
    ctx.body = '等待webpack打包完成后再访问';
    return;
  }

  // 获取最新的vue-ssr-client-manifest.json
  let clientManifestResp, clientManifest;
  try {
    clientManifestResp = await axios.get('http://localhost:8080/vue-ssr-client-manifest.json');
    clientManifest = clientManifestResp.data;
  } catch (err) {
    console.log('获取资源接口资源错误！');
  }

  const renderer = createBundleRenderer(bundle, {
    runInNewContext: false,
    template: fs.readFileSync(path.resolve(__dirname, '../src/index.temp.html'), 'utf-8'),
    clientManifest,
  });
  try {
    const html = await renderToString(ctx, renderer);
    ctx.body = html;
  } catch (err) {
    console.log('err event: ' + err);
  }
};

function renderToString(context, renderer) {
  return new Promise((resolve, reject) => {
    renderer.renderToString(context, (err, html) => {
      err ? reject(err) : resolve(html);
    });
  });
}

const router = new Router();

router.get('(.*)', handleRequest);

module.exports = router;
```
