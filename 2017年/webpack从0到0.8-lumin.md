# webpack从0到0.8
webpack是资源模块化打包构建的工具，比grunt/gulp更强大，可以将所有的静态资源当作模块处理。

**特点：**
- 静态资源模块化，一切皆模块
- 适合大型代码组织结构
- 代码分包异步加载


## 初始项目
```
$ mkdir webpack-demo && cd webpack-demo
$ npm init -y
```


这时已经生成了package.json文件，在src目录下新建一个main.js，及css，img文件夹
```javascript
console.log('this is main.js')
```

目录结构如下：
```
├── src
│   ├── main.js
│   ├── assets
│   │   ├── css
│   │   └── img
├── README.md
├── index.html
├── package.json
├── webpack.config.js
└── yarn.lock
```
## 安装

本地或者全局安装webpack

```
$ npm i webpack -D
```

**webpack.config.js配置信息**
一个webpack配置主要包括四部分信息
1. 入口（entry）：告诉webpack打包从哪里开始
2. 输出（output）：如何处理打包代码及打包输出到什么位置
3. 加载器（loaders）：通过loader识别出各种资源，将这些文件转换为模块
4. 插件（plugins）：由于加载器仅基于单个文件执行转换，插件可以做一些更复杂的操作及自定义功能

> 参考：
[http://webpack.github.io/docs/configuration.html](http://webpack.github.io/docs/configuration.html)
[https://webpack.vuefe.cn/configuration/](https://webpack.vuefe.cn/configuration/)


这里是一个最简单的配置示
```
module.exports = {
    entry: './src/main.js',
    output: {
        filename: '/dist/bundle.js'
    }
};
```

运行打包，我们可以看到这dist目录下生成了bundle.js文件
```
$ webpack
```


webpack的其他参数
- webpack -p – 进行优化压缩处理，相当于设置process.env.NODE_ENV="production"
- webpack --watch – 持续监听构建
- webpack -d – debug模式，包含source maps
- webpack --display-error-details - 显示详细的打包出错信息
- webpack -h 查看更多的信息，常见的还有--colors，--progress

## 加载更多资源 -  使用loader

先看一个例子: 把src文件夹下的.js的文件时使用babel-loader进行处理，node_modules下的文件则忽略排除
```
{
	test: /\.js$/, 
	loader: 'babel-loader', 
	include: path.resolve(__dirname, 'src'),
	exclude: /node_modules/,
}
```
loader是比较核心的一块内容，它将各类静态资源通过loader转换为js模块，一个loader包含以下几部分
```
test： 一个匹配loaders所处理的文件的拓展名的正则表达式（**必须**）
loader： loader的名称（**必须**）
include/exclude: 添加必须处理的文件（文件夹）或屏蔽不需要处理的文件（文件夹）（可选）
query： 为loaders提供额外的设置选项（可选）
```

### 1. ES6语法的js   babel-loader
现在要使用es6的语法写js，那么需要使用特定的加载器来处理这类资源，这里我们用到的是babel-loader，首先安装相应的包
```
$ npm i babel-loader -D
```
同时需要安装babel-core和babel-preset-es2015，将ES6的代码转换成ES5
```
$ npm i babel-core babel-preset-es2015 -D
```
再在module.rules中添加loader节点
```
module: {
    rules: [{
		test: /\.js$/, 
		loader: 'babel-loader', 
		query: {presets: ['es2015']}
	}]
}
```
可以将query去掉，同时新建一个**.babelrc**文件,这样方便将ES的相关配置文件都放在这个文件里面
```
{
  "presets": ["es2015"]
}
```

### 2. 样式  css-loader，style-loader
src/assets/css下新建style.css
```
body {
    background-color: #ff0;
}
```
这时js中引入这个css是会报错的，因为没有相应的loader进行处理
```
$ npm i css-loader style-loader -D
```
添加loader处理
```
{
    test: /\.css$/,
    loader: 'style-loader!css-loader'
}
```
这样css资源就可以像之前的js那样模块化的方式引入进来，main.js中添加行
```
require('./assets/css/style.css')
```
在编译的js代码中我们可以看到
	```
	/* 4 */
	/***/ (function(module, exports, __webpack_require__) {
	
	exports = module.exports = __webpack_require__(3)();
	// imports
	
	// module
	exports.push([module.i, "body {\r\n    background-color: #ff0;\r\n}", ""]);
	
	// exports
	/***/ })
	```

### 3. 图片 file-loader url-loader
url-loader是对file-loader的封装
```
$ npm i file-loader url-loader -D
```
添加loader
```
{test: /\.(png|jpg)$/, loader: "url-loader?limit=8192"}
```
这样会将小于8kb的图片直接以base64的格式内嵌到代码中，在一定程度上减少小图片的请求
main.js中添加代码
```javascript
let img1 = document.createElement('img')
img1.src = require('./assets/img/logo.png')
document.body.appendChild(img1)
```
可以看到构建后的js里以base64的格式引入了图片

> 更多的loaders：[http://webpack.github.io/docs/list-of-loaders.html](http://webpack.github.io/docs/list-of-loaders.html)


## 我需要跑个服务 webpack-dev-server
 
首先我们新建一个index.html文件
```
<!DOCTYPE html>
<html>
<head>
    <title>webpack demo page</title>
</head>
<body>
    <script src="/dist/bundle.js"></script>
</body>
</html>
```

我们需要在当前目录下跑一个服务，类似的有很多选择，使用webpack，我们可以使用它自身的devserver，如果用express，可以用webpack-dev-middlemare
```
$ npm i webpack-dev-server -D
```
在module下添加配置
```
devServer: {
	contentBase: path.join(__dirname, 'dist'),
	compress: false,
	inline: true,
	port: 8080
}
```
> 更多配置项：[http://webpack.github.io/docs/webpack-dev-server.html](http://webpack.github.io/docs/webpack-dev-server.html)


运行
```
$ node_modules/.bin/webpack-dev-server
```
浏览器查看http://127.0.0.1:8080/， 可以看到页面的背景根据我们的设置已经变成黄色

我们可以将这个脚本放在package.json中
```
"scripts": {
	"start": "node_modules/.bin/webpack",
	"serve": "node_modules/.bin/webpack-dev-server"
}
```
或者加入进程守护nodemon，安装webapck-dev-server安装到全局
```
"scripts": {
    "start": "nodemon --exec webpack -w webpack.config.js",
    "serve:watch": "nodemon  --exec webpack-dev-server -w webpack.config.js"
 }
```
这样我们就可以直接运行

```
$ npm run serve:watch
```
## 页面实时刷新 Hot Module Replacement(HMR) 热加载
> It’s like LiveReload for every module.
当模块发生变化时，内存中的bundle会收到通知，如果不影响到整个页面的变化，只会刷新局部，而不用刷新整个页面。

1. 设置--hot为true
```
devServer: {
    contentBase: path.join(__dirname, ''),
    compress: true,
    inline: true,
    hot: true, 
    port: 8080
},
```

2. 添加到plugins中
```
new webpack.HotModuleReplacementPlugin() //热加载插件
```

3. chrome控制台打开我们可以看到，说明HMR已经开启
```
[HMR] Waiting for update signal from WDS...            dev-server.js:49 
[WDS] Hot Module Replacement enabled.                  client?d3ca:41
```

> 参考：[https://github.com/glenjamin/webpack-hot-middleware](https://github.com/glenjamin/webpack-hot-middleware)

## 附：webpack修改每次都重启？nodemon守护
nodemon进程守护，用来监控你node.js源代码的任何变化，自动重启服务
```
$ npm i nodemon -g
$ nodemon --exec webpack -w webpack.config.js
```
> 这里我们只需要监听webpack.config.js文件的变化，所以添加-w参数指定特定的目录或者文件 


到这里一个基本的wepack构建过程已经完成，但是webpack功能好像我们只用了一部分，让我们一起来看看哪些可以更完善。

---

## 进阶
目录结构如下：
```
├── src
│   ├── main.js				--主入口
│   ├── assets
│   │   ├── css
│   │   └── img
│   ├── libs
│   │   └── util.js			--公共方法
│   ├── modules
│   │   ├── login.js      	--登录页面
│   │   └── product.js		--商品页面
├── README.md
├── index.html
├── package.json
├── webpack.config.js
└── yarn.lock
```
### 1. 多页应用
entry的配置我们可以是string，object，array类型，前面的例子用到的是string，单个入口，现在我们添加了几个目录及文件
```
entry: {
    main: './src/main.js',
    login: './src/modules/login.js',
    product: './src/modules/product.js'
}
```
上面的入口文件都在src目录下，那么可以设置一个基础目录，绝对路径，用于从配置中解析入口起点(entry point)和加载器(loader)
```
context: path.resolve(__dirname, 'src'),
entry: {
    main: './main.js',
    login: './modules/login.js',
    product: './modules/product.js'
}
```
如果是数组，那么会将数组中的模块合并，并且输出最后一个；如果是object，那么多个入口的key会打包成包的chunk名称。
```
output: {
    path: path.join(__dirname, 'dist'),
    publicPath: '/',
    filename: '[name]-[hash:8].js',
    chunkFilename: '[id]-[chunkhash].js'
}
```

但是，新加一个入口就要手动添加？可以设置一定规则，用nodejs遍历自动生成入口。

### 2. 省略文件扩展名？resolve.extensions
可以直接写util，而不用util.js， vue文件也可以省略文件名
```
import {ajax} from "./libs/util"
import one from './one' // one.vue
```
```
resolve: {
    extensions:  ['', '.js', '.vue'] 
}
```
### 3. 文件查找的路径太长？resolve.alias缩减引用路径
```
resolve: {
    extensions:  ['.js', '.css'] ,
    alias: {
        'libs': path.resolve(__dirname, 'src/libs')，
		'react': 'node_modules/react/react.js'
    }
}
```
这样在src下的任何js文件都可以直接这样引入模块，而不用../libs/util
```
import { ajax } from 'libs/util'
```
### 4. 自动引入vue/jquery ？ProvidePlugin
自动加载模块，ProvidePlugin可以让我们无需引入的情况下，以全局的模式直接使用模块变量
```
new webpack.ProvidePlugin({
    Vue: 'Vue'
})
```
代码中可以不引入Vue而直接使用Vue

### 5. 全局的常量 **DefinePlugin**
允许你创建一个在编译时可以配置的全局常量。这可能会对开发模式和发布模式的构建允许不同的行为非常有用。比如,你可能会用一个全局的常量来决定 log 在开发模式触发而不是发布模式。这仅仅是 DefinePlugin 提供的便利的一个场景。
```
new webpack.DefinePlugin({
    'process.env': {
        NODE_ENV: '"production"'
    }
}),
new webpack.DefinePlugin({
  'NICE_FEATURE': JSON.stringify('this is global defined'),
  'EXPERIMENTAL_FEATURE': JSON.stringify(false)
})```

> 注意这里的JSON.stringify，因为DefinePlugin是直接找到到变量将内容替换，如果不加这个，替换的内容会没有引号而报错


### 6. 公用代码的处理：封装组件 **CommonsChunkPlugin**
将多个入口起点之间共享的公共模块，生成为一些 chunk，并且分离到单独的 bundle 中，例如，1vendor.bundle.js 和 app.bundle.js
```
new webpack.optimize.CommonsChunkPlugin({
    name: 'commons',
    // ( 公共chunk(commnons chunk) 的名称)

    filename: 'commons-[hash].js',
    // ( 公共chunk 的文件名)

    minChunks: 3,
    // (模块必须被3个 入口chunk 共享)

    // chunks: ['pageA', 'pageB'],
    // (只使用这些 入口chunk)
}),
```

### 7. 引用外部的CDN资源？externals
不管是使用resolve.alias还是ProvidePlugin，打包的时候，webpack都会将使用到的库进行打包。如果我们不想webpack打包某个资源，而是直接在页面使用标签手动引入，或者使用CDN资源的时候，externals这个配置项就起作用了。
在module节点下添加
```
externals: {
    $: 'window.jQuery'
}
```

### 8. 打包单独css： ExtractTextWebpackPlugin
从 bundle 中提取文本（CSS）到分离的文件（app.bundle.css）
```
{loader: ExtractTextPlugin.extract({fallback: 'style-loader', use:'css-loader'})}
```
plugins添加:
```
new ExtractTextPlugin('[name]-[hash].css'),
```
### 9. 压缩代码： UglifyjsWebpackPlugin
这个插件使用 UglifyJS 去压缩你的JavaScript代码，内置插件，因为很耗性能，所以只在发布上线时使用
```
new webpack.optimize.UglifyJsPlugin({
    compress: {
        warnings: false
    }
})
```

## 其他插件 plugins
**BannerPlugin**
为每个 chunk 文件头部添加 banner
```
new webpack.BannerPlugin('Copyright aiyoumi.com inc.')
```
 

**HtmlWebpackPlugin**
用于简化 HTML 文件（index.html）的创建，提供访问 bundle 的服务。
```
new HtmlWebpackPlugin({
    template: path.join(__dirname + '/index.html')
})
```

**NormalModuleReplacementPlugin**
替换与正则表达式匹配的资源
 
**clean-webpack-plugin**
编译前清空指定的文件夹
```
new CleanWebpackPlugin(['dist', 'build']),
```

**DllPlugin**
将第三方引入的不经常变动的代码打包成一个vendor，页面中单独引入这个vendor

**DllReferencePlugin**
将DllPlugin生成的公共代码通过生成的mainfest.json引入

**webpack-browser-plugin**
自动打开浏览器


> **更多插件plugins** [http://webpack.github.io/docs/list-of-plugins.html](http://webpack.github.io/docs/list-of-plugins.html)

## 调试 devtool
webpack打包构建的代码不利于调试，如果想调试源代码，可以使用devtool，像vue，react等框架都会有对应的devtools，方便调试
- devtool: "source-map", // 这是最原始的 source-map 实现方式，其实现是打包代码同时创建一个新的 sourcemap 文件， 并在打包文件的末尾添加 //# sourceURL 注释行告诉 JS 引擎文件在哪儿
- devtool: "inline-source-map", // 将 SourceMap 以Base64 格式化后的字符串嵌入到源文件中
- devtool: "eval-source-map", // 把 eval 的 sourceURL 换成了完整 SourceMap 信息的 DataUrl
- devtool: "hidden-source-map", // SourceMap .map文件结尾不显示
- devtool: "cheap-source-map", // 没有模块映射(module mappings) 不包含列信息，不包含 loader 的 sourcemap
- devtool: "cheap-module-source-map", // 不包含列信息，同时 loader 的 sourcemap 也被简化为只包含对应行的。
- devtool: "eval", // 每个模块都封装到 eval 包裹起来，并在后面添加 //# sourceURL


> 参考：[http://webpack.github.io/docs/configuration.html#devtool](http://webpack.github.io/docs/configuration.html#devtool)


## webpack2
以上代码及示例是基于webpack1版本下的，webpack2有不少改变及性能上的提升，像tree-shaking，我们可以对比一下jsnpm，rollup等。

下一篇将介绍基于webpack2.0的项目实战总结及优化经验分享。