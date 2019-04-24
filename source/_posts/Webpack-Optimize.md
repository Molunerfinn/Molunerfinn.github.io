---
title: Vuejs项目的Webpack2构建优化
tags: 
  - 前端
  - Webpack
categories:
  - Web
  - 开发
date: 2017-05-25 11:35:00
---

最近在做的项目因为相对较大（打包有100多个chunk），在build构建的时候速度一直上不去，甚是烦恼。由于用的是`vue-cli`的`webpack2`模板，一开始并没有想着要对其进行优化，一直觉得是webpack本身慢+硬件慢（在开发机上开发，内存和CPU都不是很强力）的原因。后来慢到实在受不了了，转移到本地（i7+16G）开发的时候，发现生产构建居然需要90s，实在太长了。所以开始着手Webpack2构建优化。

<!-- more -->

优化webpack构建速度，总的来说有几个思路：

1. 优化本身项目结构，模块的引入、拆分、共用
2. 优化结构路径、让webpack接管的文件能够快速定位
3. 优化uglify编译速度
4. 优化webpack本身编译速度

有些是在开发的时候代码层面上的，有些则是需要在webpack配置层面上的。对于开发层面上来说，按需引入是很重要的一点。通常为了方便我们可以直接引入一个`echarts`，但是实际上并不需要`echarts`的所有功能。而按需引入则能最大程度上让

总得来说作用最大的有这几个：

- 开启`webpack`的cache
- 开启`babel-loader`的cache
- 指定`modules`以及配置项目相关的`alias`
- 配置`loader`的`include`和`exclude`
- 用`CommonsChunkPlugin`提取公用模块
- 使用`DllPlugin`和`DllReferencePlugin`预编译
- 换用`happypack`多进程构建
- `css-loader`换成`0.14.5`版本。
- 换用`webpack-uglify-parallel`并行压缩代码

以下的配置都是基于`vue-cli`的`webpack`模板进行的优化。

### 开启webpack的cache

打开`webpack.base.conf.js`，在`module.exports`里加上`cache: true`：

```js
module.exports = {
  cache: true,
  // ... 其他配置
}
```

### 开启`babel-loader`的cache

开启了cache的`babel-loader`，在下次编译的时候，遇到不变的部分可以直接拿取cache里的内容，能够较为明显地提高构建速度。在`loader`选项里只需要对`babel-loader`开启`cacheDirectory=true`即可。

```js
// ... 其他配置
module: {
  rules: [
    {
      test: /\.js$/,
      loader: ['babel-loader?cacheDirectory=true']
    },
    // ... 其他loader
  ]
}

```

### 配置`modules`以及配置项目相关的`alias`

这个部分的配置实际上都是对webpack接管的文件路径的一些配置。通过这些配置，webpack可以不必自己遍历去搜索模块等，而可以通过我们定义的路径，快速定位。尤其是`node_modules`的位置，这个位置可以通过`modules`选项配置，节省webpack去查找的时间。

而`alias`是别名。通过编写`alias`，既能让webpack查找文件定位更快，在开发的时候，也能少些很多相对路径的`../..`，在引入模块的时候很方便。

同样是打开`webpack.base.conf.js`，在`module.exports`的`resolve`属性里配置`modules`和`alias`。其中`vue-cli`会自动配置一些默认的`alias`。

```js
resolve: {
  //... 其他配置
  modules: [path.resolve(__dirname, '../../node_modules')], // node_modules文件夹所在的位置取决于跟webpack.base.conf.js相对的路径
  alias: {
    //... 其他配置
    api: path.resolve(__dirname, '../../server/api') // api文件所在的位置取决于跟webpack.base.conf.js相对的路径，在项目中会自动转换跟项目文件的相对路径
    //... 其他配置
  }
}
```

如果配置了如上的`alias`，那么我们在项目里，要引用比如`api.js`这个模块，可以直接这样做了：

```js

import * as api from 'api' // 'api'是个alias，webpack会直接去找`server/api`

```

而不用手动去根据项目文件和api所在路径的相对位置去书写import的路径了。

### 配置`loader`的`include`和`exclude`

`loader`的`include`和`exclude`也就是需要`loader`接管编译的文件和不需要`loader`接管编译的文件。

这里我们举`babel-loader`为例。通常情况下，我们不需要`loader`去编译`node_modules`下的js文件，而我们只需要编译我们项目目录下的`js`就行了。这样可以通过配置这两个选项，能够最小范围的限制`babel-loader`需要编译的内容，能够有效提升构建速度。

同样打开`webpack.base.conf.js`，在`rules`的`babel-loader`那块加上`include`和`exclude`。

```js
// ... 其他配置
module: {
  rules: [
    {
      test: /\.js$/,
      loader: ['babel-loader?cacheDirectory=true'],
      include: [resolve('src')], // src是项目开发的目录
      exclude: [path.resolve('../../node_modules')] // 不需要编译node_modules下的js 
    },
    // ... 其他loader
  ]
}
```

### 使用`CommonsChunkPlugin`提取公用模块

我们经常会有这种场景：在`a.vue`组件里引入了`a.js`或者比如`c.vue`，在`b.vue`组件里也引入了`a.js`或者`c.vue`。这样，打包了之后将会把引入的模块重复打包。而`CommonsChuncksPlugin`就是把这样重复打包的模块给抽取出来单独打包的插件。这个能够显著降低最后打包的体积，也能提升一些打包速度。

在`webpack.base.conf.js`里的`plugins`可以加上这段：

```js
plugins: [
  new webpack.optimize.CommonsChunkPlugin({
    async: 'shared-module',
    minChunks: (module, count) => (
      count >= 2    // 当一个模块被重复引用2次或以上的时候单独打包起来。 
    )
  }),

  //...
]

```


### 使用`DllPlugin`和`DllReferencePlugin`预编译

这个也是一个大杀器。将一些全局都要用到的依赖抽离出来，预编译一遍，然后引入项目中，作为依赖项。而webpack在正式编译的时候就可以放过他们了。能够很明显地提升webpack的构建速度。类似于Windows的`dll`文件的设计理念。dll资源能够有效的解决资源循环依赖的问题。能够大大减少项目里重复依赖的问题。

在`webpack.base.conf.js`所在的文件夹里建立一个`webpack.dll.conf.js`，我们将一些常用的依赖打包成`dll`。

首先配置一下`DllPlugin`的资源列表。

```js
const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: {
    vendor: ['vue/dist/vue.esm.js','vue-router','axios','vuex'] // 需要打包起来的依赖
  },
  output: {
    path: path.join(__dirname, '../../public/js'), // 输出的路径
    filename: '[name].dll.js', // 输出的文件，将会根据entry命名为vendor.dll.js
    library: '[name]_library' // 暴露出的全局变量名
  },
  plugins: [
    new webpack.DllPlugin({
      path: path.join(__dirname, '../../public/js/', '[name]-mainfest.json'), // 描述依赖对应关系的json文件
      name: '[name]_library', 
      context: __dirname // 执行的上下文环境，对之后DllReferencePlugin有用
    }),
    new webpack.optimize.UglifyJsPlugin({ // uglifjs压缩
      compress: {
        warnings: false
      }
    })
  ]
}
```

为了方便之后构建，可以在`package.json`里加上这么一句`scripts`：

```js
scripts: {
  //... 其他scripts
  "build:dll": "webpack --config ./webpack/build/webpack.dll.conf.js" // 填写你项目中webpack.dll.conf.js的路径
}
```

然后执行一下`npm run build:dll`，就可以在输出的目录里输出一个`vendor.dll.js`和`vendor-mainfest.json`两个文件。

之后打开`webpack.base.conf.js`。在plugins一项里加上`DllReferencePlugin`。这个plugin是用于引入上一层里生成的json的。

```js
module.exports = {
  //... 其他配置
  
  plugins: [
    // ... 其他插件
    new webpack.DllReferencePlugin({
      context: __dirname,
      manifest: require('../../public/js/vendor-mainfest.json') // 指向这个json
    })
  ]
}

```

最后，在项目输出的`index.html`里，最先引入这个js：

```html
<script type="text/javascript" src="public/js/vendor.dll.js"></script>
```

这样，webpack将不会再解析dll里的资源了。构建速度将会有质的提高。

### 换用`happypack`多进程构建

webpack的构建毕竟还是单进程的。采用`happypack`可以改为多进程构建。而对于小文件而言，`happypack`效果并不明显。而对于`babel-loader`编译的庞大的js文件群来说，则是一大利器。

首先安装：`npm install happypack --save-dev`或者`yarn add happypack`

然后修改`webpack.base.conf.js`的配置如下：

```js
const os = require('os');
const HappyPack  = require('happypack');
const happThreadPool = HappyPack.ThreadPool({size: os.cpus().length}); // 采用多进程，进程数由CPU核数决定

//...
module.exports = {
  plugins: [
    // ...
    new HappyPack({
      id: 'js',
      cache: true,
      loaders: ['babel-loader?cacheDirectory=true'],
      threadPool: happThreadPool
    })
  ],
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {
          loaders: {
            js: 'happypack/loader?id=js' // 将loader换成happypack
          }
        }
      },
      {
        test: /\.js$/,
        loader: ['happypack/loader?id=js'], // 将loader换成happypack
        include: [resolve('src')], // src是项目开发的目录
        exclude: [path.resolve('../../node_modules')] // 不需要编译node_modules下的js 
      },
      //...
    ]
  }
}
```

提速还是比较明显的。

### `css-loader`换成`0.14.5`版本

可以查看这个[issue](https://github.com/webpack-contrib/css-loader/issues/124)，说是该版本之上的版本会拖慢webpack的构建速度。我自己实验了之后确实能快几秒钟。

### 换用`webpack-uglify-parallel`并行压缩代码

webpack自带的uglifyjs插件效果确实不错。只不过由于受限于单线程，所以压缩速度不够高。换成`webpack-uglify-parallel`这个插件之后能够有效减少压缩的时间。

首先安装：`npm install webpack-uglify-parallel --save-dev` 或者 `yarn add webpack-uglify-parallel`

找到`webpack.prod.conf.js`（由于开发模式不需要进行uglify压缩），将原本的：

```js
plugins: [
  new webpack.optimize.UglifyJsPlugin({
    compress: {
      warnings: false
    },
    sourceMap: true
  })
  // ... 其他配置
]
```

替换为：

```js
const UglifyJsparallelPlugin = require('webpack-uglify-parallel');
const os = require('os');

// ... 其他配置
plugins: []
new UglifyJsparallelPlugin({
  workers: os.cpus().length,
  mangle: true,
  compressor: {
    warnings: false,
    drop_console: true,
    drop_debugger: true
  }
})
```

-------

### 效果综述

经过以上优化，原本90+s的构建时间，优化到30+秒效果还是很明显的。原本HMR模式初始化需要50+秒，也优化到了20+秒的程度。不过，还是有一个至今我自己还无法解决的问题，那就是HMR模式，rebuild时间会越来越长，直到超过初始化的20+秒。此时只能重新开关一次HMR模式。这一点我还无法找到具体原因是什么。不过，至少生产的构建时间得到了60%的优化，效果还是挺好的。
