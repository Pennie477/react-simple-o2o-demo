# 搭建 webpack + React 开发环境

使用webpack 搭建 React 开发环境
[参考] (https://juejin.cn/post/6844904079219490830)

## 开始之前

- 使用的系统是 mac os 系统
- windows有模拟 linux 命令的工具，例如 `xshell`

## 初始化 npm 环境并安装插件

使用 npm 管理第三方依赖（插件）

### 初始化 npm 环境

首先保证有 node 和 npm 环境，运行`node -v`和`npm -v`查看
进入项目目录，运行`npm init` 最终生成`package.json`文件

### 安装插件

已知我们将使用 webpack 作为构建工具，那么就需要安装相应插件，运行 `npm install webpack webpack-dev-server --save-dev` 来安装两个插件。

又已知我们将使用 React ，也需要安装相应插件，运行 `npm i react react-dom --save`来安装两个插件。

安装完成之后，查看`package.json`可看到多了`devDependencies`和`dependencies`两项，根目录也多了一个`node_modules`文件夹。

## 配置 webpack.config.js

webpack工具基于该文件进行项目打包构建

### 文件格式

webpack.config.js 就是一个普通的 js 文件，符合 commonJS 规范。最后输出一个对象，即`module.exports = {...}`

### 输入 & 输出

- 需要新建`./app/index.jsx`作为入口文件，目前项目中只有这一个入口文件。不过 webpack 支持多个入口文件，可查阅文档。

- 输出就一个`bundle.js`文件，输出的 js 和 css 都在里面
不过只有在开发环境下才用，发布代码的时候，肯定不能只有这么一个文件。

### module

针对不同类型的文件，使用不同的`loader`，当然使用之前要安装，例如`npm i style-loader css-loader --save-dev`。该项目代码中，我们用到的文件格式有：js/jsx 代码、css/less 代码、图片、字体文件。

less 是 css 的语法糖，可以更高效低冗余的写 css

在加载 css/less 时用到了`postcss`，主要使用`autoprefixer`功能，帮助自动加 css3 的浏览器前缀，非常好用。

编译 es6 和 jsx 语法时，用到家喻户晓的`babel`，另外还需增加一个`.babelrc`的配置文件。

### plugins

使用 html 模板（需要`npm i html-webpack-plugin --save-dev`），这样可以将输出的文件名（如`./bundle.js`）自动注入到 html 中，不用我们自己手写。
 手写的话，一旦修改就需要改两个地方。

使用热加载和自动打开浏览器插件

### devServer

对 webpack-dev-server 的配置

### 配置package.json

`process.env`中默认并没有`NODE_ENV`，需要配置下`package.json`。

- 为了兼容Windows和Mac，需安装 `cross-env`:
`npm install cross-env -D`

- 在`package.json`的 `scripts` 中，输入以下代码，将这两串命令分别封装为`npm run dev`和`npm run build`，这样就可以运行项目代码了。
(运行前先`cnpm i`确保依赖都搭建好)

```json
  "scripts": {
    "dev": "cross-env NODE_ENV=dev webpack-dev-server",
    "build": "cross-env NODE_ENV=production webpack"
},
```

代码中`NODE_ENV=dev`代表当前是开发环境下，这里的`"dev"`可被 js 代码中的`process.env.NODE_ENV`得到并做一些其他处理。

### 定义环境全局变量

以下定义，可使得代码通过`__DEV__`得到当前是不是开发模式。

```js
new webpack.DefinePlugin({
  __DEV__: JSON.stringify(JSON.parse(process.env.NODE_ENV == "dev" || "false")),
});
```

打开`./app/util/localStore.js`可以看到`if (__DEV__) { console.error('localStorage.getItem报错, ', ex.message) }`，即只有开发环境下才提示 error，发布之后就不会提示了。（因为发布的命令中用到`NODE_ENV=production`）

## 配置 webpack.production.config.js

开发环境下，可以不用考虑系统的性能，更多考虑的是如何增加开发效率。而发布系统时，就需要考虑发布之后的系统的性能，包括加载速度、缓存等。

### 发布到 `./build` 文件夹中

`path: __dirname + "/build",`

### vendor

将第三方依赖单独打包。即将 node_modules 文件夹中的代码打包为 vendor.js 将我们自己写的业务代码打包为 app.js。这样有助于缓存，因为在项目维护过程中，第三方依赖不经常变化，而业务代码会经常变化。

### md5 后缀

为每个打包出来的文件都加 md5 后缀，例如`"/js/[name].[chunkhash:8].js"`，可使文件强缓存。

### 分目录

打包出来的不同类型的文件，放在不同目录下，例如图片文件将放在`img/`目录下

### Copyright

自动为打包出来的代码增加 copyright 内容

### 分模块

`new webpack.optimize.OccurenceOrderPlugin(),`

### 压缩代码

使用 Uglify 压缩代码，其中`warnings: false`是去掉代码中的 warning

### 分离 css 和 js 文件

开发环境下，css 代码是放在整个打包出来的那个 bundle.js 文件中的，发布环境下当然不能混淆在一起，使用`new ExtractTextPlugin('/css/[name].[chunkhash:8].css'),`将 css 代码分离出来。

### npm run build

`process.env` 中默认并没有 `NODE_ENV`，这里配置下我们的 `package.json` 的 `scripts`.

为了兼容Windows和Mac，先安装一下 `cross-env`:
`npm install cross-env -D`

打开`package.json`，查看以下代码。`npm run dev`和`npm run build`分别是运行代码和打包项目。对比下 `dist/index.html` ，可以看到 `npm run build`，生成的 `index.html` 文件中引入了对应的 `css` 和 `js`。且对应的 `title` 内容也不一样

```json
  "scripts": {
    "dev": "cross-env NODE_ENV=development webpack",
    "build": "cross-env NODE_ENV=production webpack"
  },
```

### 最小化压缩 React

以下配置可以告诉 React 当前是生产环境，请最小化压缩 js ，即把开发环境中的一些提示、警告、判断通通去掉，只留以下发布之后可用的代码。

```js
    new webpack.DefinePlugin({
      'process.env':{
        'NODE_ENV': JSON.stringify(process.env.NODE_ENV)
      }
    }),
```
