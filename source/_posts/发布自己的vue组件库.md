---
title: 发布自己的vue组件库
categories: [技术]
date: 2017-10-13 12:56:12
tags: [vue, javascript, npm]
---

## 目标

总结一下最近练习的如何使用npm发布属于自己的vue组件库。虽然不成熟且十分简单，但是还是有收获的，后期会一点一点完善。

那么接下来的操作都按步骤来：（假设npm包取名cdcomponents）

### vue-cli新建项目

```shell
vue init cdcomponents
cd cdcomponents
npm i
```

<!--more-->

### 修改一些配置

**1. 进入config/index.js, 将其中的build对象修改如下：**

```js
build: {
    env: require('./prod.env'),
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsPublicPath: '/',
    assetsSubDirectory: '/',
    productionSourceMap: true,
    // Gzip off by default as many popular static hosts such as
    // Surge or Netlify already gzip all static assets for you.
    // Before setting to `true`, make sure to:
    // npm install --save-dev compression-webpack-plugin
    productionGzip: false,
    productionGzipExtensions: ['js', 'css'],
    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report
  }
```

这样打包的时候只会将入口文件的信息打包成一个js文件和一个css放到dist目录下面。

**2. 修改webpack.prod.config.js下面的output参数，并删除一些没用的配置，最终修改如下：**

```js
var path = require('path')
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var CopyWebpackPlugin = require('copy-webpack-plugin')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')

var env = config.build.env

var webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  devtool: config.build.productionSourceMap ? '#source-map' : false,
  output: {
    path: config.build.assetsRoot,
    publicPath: config.build.assetsPublicPath,
    filename: 'cdcomponents.min.js',
    library: 'CdComponents',
    libraryTarget: 'umd'
  },
  plugins: [
    // http://vuejs.github.io/vue-loader/en/workflow/production.html
    new webpack.DefinePlugin({
      'process.env': env
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      sourceMap: true
    }),
    // extract css into its own file
    new ExtractTextPlugin({
      filename: 'cdcomponents.min.css'
    }),
    new OptimizeCSSPlugin()
  ]
})

module.exports = webpackConfig
```

### 开始写组件

组件的具体实现过程我就不阐述了，最终的src目录和packages目录结构如下：

其中packages放置各个组件，src里面的main.js进行组件的插件化处理

![](http://oifeo8q69.bkt.clouddn.com/npmpost1.jpg)

packages/rating/index.js文件如下，这里加了install方法是为了后面留出口的时候，用户使用可以直接引入单组件，而不必引入整个组件库

```js
import Rating from './src/rating.vue';

/* istanbul ignore next */
Rating.install = function(Vue) {
  Vue.component(Rating.name, Rating);
};

export default Rating;
```

后续使用单组件就可以类似这样实现：

```js
import { Rating } from 'cdcomponents';

此处省略...

export default new Vue({
  el: '#app',
  template: '<Rating />',
  components: { Rating }
});

```

packages/rating/src/rating.vue文件， 组件我就不写了，主要实现了五星评价的功能。

src/main.js

```js
// import Vue from 'vue';
// import App from './App.vue';
import Rating from '../packages/ratings';

const components = [
  Rating
];
const install = function(vue) {
  /* istanbul ignore if */
  if (install.installed) return;

  /*eslint-disable*/
  components.map((component) => {
    vue.component(component.name, component);
  });
};

/* istanbul ignore if */
if (typeof window !== 'undefined' && window.Vue) {
  install(window.Vue);
};
// 测试的时候加上注释
export default {
  install,
  Rating
};

// 测试的时候打开注释
// Vue.use(install);
//
// export default new Vue({
//   el: '#app',
//   template: '<App/>',
//   components: { App }
// });
```

至此，组件的组件的注册插件都写好了，最后一步就是稍微修改一下package.json文件：
因为这个组件包是能公用的，所以"private": false

```js
"private": false,
"main": "dist/cdcomponents.min.js",
```

这里的main配置意思是，别人用这个包 import CdComponents from 'cdcomponents'; 时，引入的文件。
至此，你的npm包就可以发布到网上，当别人npm i --save cdcomponents以后，在项目中可以使用以下方式引入：

```js
import CdComponents from 'cdcomponents'; // 默认引入 dist/cdcomponents.min.js
import 'cdcomponents/dist/cdcomponents.min.css';

Vue.use(CdComponents);
```

### npm 发布

首先在本地执行`npm adduser`,如果有账号就直接输入用户名密码登录，如果没有要先去官网注册一下账号。

然后打包本地文件，`npm run build`，这时根目录下就会出现dist文件夹，里面包含了打包好的js和css文件。

然后执行`npm publish`就可以发布到npm上面啦！大功告成

如果后期需要更新包，那就执行

```shell
npm run build
npm version patch //小版本更新
npm publish

想要删除包
npm unpublish --force
```

### 最后

[github地址](https://github.com/zp1112/cdcomponents)
[npm地址](https://www.npmjs.com/package/cdcomponents)