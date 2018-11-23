# webpack进阶与优化实践

本文是基于上一篇的进阶，基于webpack2.0对爱又米vue项目进行构建优化的实践总结。

## 对比
先来看看最直观的优化前后的对比图

***优化前（基于webpack 1.14.0版本）***
![old](https://img.aiyoumi.com/null/201746/163833136/20170406163832_1602x771.png@80q)

***优化后（基于webpack 2.3.2版本）***
![new](https://img.aiyoumi.com/null/201746/164624306/20170406164624_1804x710.png@80q)

可以明显看出构建的速度有了明显的提升，从原来的40多秒提高到了10秒不到，快了**3倍**，当然最后基于实际的项目需要，用了dll方式，而没有选择external方式，最后的速度也是在12秒左右，加上dll构建，共14秒不到，也是快了**2倍**多，优化结果还是很可观的。

 
## 分析
在这个过程中我们做了哪些优化？首先是分析哪些环节是比较费时间的，然后针对比较耗时的部分进行优化。通过profile以及一些测试对比，发现了这几部分是可以进行优化的
- 脚本压缩部分
- 一些包的重复构建
- 选择合适的sourcemap


相应的解决思路是：
- 多进程多任务处理
- 能缓存的尽量缓存，只构建新的资源
- 一些不常变的包抽取成公共的只编译一次或者外部引用

其中最明显的效率提升是使用多进程处理多个loader和uglify 脚本，在使用第三方插件时，不论使用哪种优化方案，一般我们都开启cache功能，这样会大大加快二次构建的速度。

## 多进程处理脚本压缩
webpack-parallel-uglify-plugin这个插件（或类似的多进程处理插件）的效果最明显，与不用uglify的时间差不多，所以只在构建到线上环境才使用压缩丑化脚本
```js
var ParallelUglifyPlugin = require('webpack-parallel-uglify-plugin')
...
plugins: [
	new ParallelUglifyPlugin({
        cacheDir: path.join(__dirname, '../.cache'), 
        workerCount: 3, // os.cpus().length, 
        uglifyJS: {
            compress: {
                warnings: false
            },
            sourceMap: true
        }
    }),
	...
]

```
## DllPlugin 和 DllReferencePlugin
实际的开发中我们通常会引入很多第三方 NPM 包，比如vue，vue-router等，这些包我们不会进行修改，但是每次 build 的过程中仍然会消耗构建性能，怎么办呢？DllPlugin 就是一个解决方案，通过一次性单独构建这些依赖包，在业务代码中就不再需要重复构建，这样来提高真正的 build 和 rebuild 的构建效率。

首先创建一个dll的配置项
```js
var path = require('path');
var webpack = require('webpack');
var config = require('./path.js');

var outputPath = process.env.NODE_ENV === 'production' ? config.build.assetsRoot : config.dev.assetsRoot;

const vendors = [
    'vue',
    'vue-router',
    'vue-resource'
]

module.exports = {
    output: {
        path: outputPath,
        filename: '[name].js',
        library: '[name]'
    },
    entry: {
        'vendors': vendors
    },
    plugins: [
        new webpack.DllPlugin({
            path: './manifest.json',
            name: '[name]',
            context: __dirname
        })
    ]
}
```
然后在 package.json 的scripts中配置命令
```
	"dll:build": "cross-env NODE_ENV=production webpack --config config/webpack.dll.config.js",
```
执行npm run dll:build，会生成vendor.dll.js 和 manifest.json

接下来就在 webpack.prof.config.js 中通过 DLLReferencePlugin 来使用 DllPlugin 生成的 DLL Bundle
```js
plugins: [
	...
	new webpack.DllReferencePlugin({
	    context: __dirname,
	    manifest: require('../manifest.json'),
	})
]
```
> Tips: 这里有个问题，当多人协作时由于每个人的npm包版本不一致，会导致的构建的Dll Bundle及manifest.json不同，这时需要个人再次编译dll:build，避免版本不一致出现的js报错。另外一种方案是统一使用yarn包管理，保证团队使用的包版本一致。


### 和 externals 对比
配置
```js
externals: {
    'vue': 'Vue',
    'vue-router': 'VueRouter',
    'vue-resource': 'VueResource'
}
```
经过测试 externals 的方式构建速度更快，至少是省去了dll的构建时间。但是有几个缺点：
- 像vue 这种已经打好了生产包的使用 externals 很方便，但是也有很多 npm 包是没有提供的，这种情况下 Dll Bundle 方式仍可以使用
- 如果只是引入 npm 包一部分的功能，比如 require('lodash/fp/extend') ，这种情况下 Dll Bundle 方式仍可以使用
- Dll Bundle 方式可以更容易扩展，比如下次再增加一个第三方库时，只需要修改dll config的配置即可，页面引用的还是生成的vendors js
- 当然如果只是引用了 vue 这类的话，externals 因为配置简单所以也推荐使用，而且可以通过把vue js库放在cdn上，加快访问速度

## happypack多线程处理
happypack 是 webpack 的一个插件，通过多进程处理来加速构建，利用此插件效果显著。
先来看看最基本的用法
```
npm install --save-dev happypack
```

```js
var HappyPack = require('happypack');
var happyThreadPool = HappyPack.ThreadPool({ size: os.cpus().length });

...

module: {
	rules: [{
            test: /\.js$/,
            loader: 'happypack/loader?id=js',
            include: [resolve('src')]
        }
	]
},
plugins: [
	new HappyPack({
		id: 'js',
		loaders: ['babel-loader'],
		threadPool: happyThreadPool,
		cache: true,
		verbose: true
	})
]
```

但是，我们的项目基于vue，happypack并没有相应的vue-loader，这样就没法使用多进程构建vue了？尤大已经提了PR，后续可以直接使用vue-loader，当前方法还是有的，我们可以把vue-loader内部拆分出独立的js,css,sass来处理。
```js
{
    test: /\.vue$/,
    loader: 'vue-loader',
    include: [resolve('src')],
    options: {
		loaders:{ 
			css: [ 'vue-style-loader', {
		        loader: 'happypack/loader?id=css'
		    }],
			sass: [ 'vue-style-loader', {
		        loader: 'happypack/loader?id=sass'
		    }, {
		        loader: 'happypack/loader?id=css'
		    } ],
			js: 'happypack/loader?id=js' 
		}
	}，
	{
        test: /\.js$/,
        loader: 'happypack/loader?id=js',
        include: [resolve('src')]
    }
}
```
添加happypack插件处理
```js
var HappyPack = require('happypack');
var happyThreadPool = HappyPack.ThreadPool({
    size: 4
});
function happyPack(type, loader) {
    return new HappyPack({
        id: type,
        cache: true,
        debug: true,
        threadPool: happyThreadPool,
        loaders: [loader]
    })
}

...

plugins: [
	...
    happyPack('js', 'babel-loader'),
    happyPack('eslint', 'eslint-loader'),
    happyPack('css', 'css-loader'),
    happyPack('sass', 'sass-loader')
]
```

官方的测试结果对比：
![http://img.aiyoumi.com/static/img/201704/12111017816.png](http://img.aiyoumi.com/static/img/201704/12111017816.png)

> happypack原理分析： http://taobaofed.org/blog/2016/12/08/happypack-source-code-analysis/


## 构建profile分析
通过分析webpack构建生成的profile文件，我们可以清楚的查看包的依赖关系，大小。

### 1. 官方分析工具 http://webpack.github.io/analyse
```
$ webpack --config build/webpack.prod.conf.js  --profile --json > stats.json
```
 把stats.json文件上传到 http://webpack.github.io/analyse 进行分析，就能看到文章开头那两张对比图。
 
下面是各个模块的chunk ID，大小，以及依赖关系等信息
![http://img.aiyoumi.com/static/img/201704/10195024610.png](http://img.aiyoumi.com/static/img/201704/10195024610.png)

![http://img.aiyoumi.com/static/img/201704/10195248417.png](http://img.aiyoumi.com/static/img/201704/10195248417.png)

这个图片可以看出模块被哪些模块引用了，引用了多少次，这时我们可以通过CommonsChunkPlugin把这公共依赖的包打包到common js中。
![http://img.aiyoumi.com/static/img/201704/10195428579.png](http://img.aiyoumi.com/static/img/201704/10195428579.png)

### 2. Webpack Chart https://alexkuz.github.io/webpack-chart/

![http://img.aiyoumi.com/static/img/201704/10200036936.png](http://img.aiyoumi.com/static/img/201704/10200036936.png)




### 3. webpack-bundle-analyzer

```
$ npm i webpack-bundle-analyzer -D
```
使用插件
```js
if (config.build.bundleAnalyzerReport) {
    var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
    webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}
```
```
$ npm run build --report
```
会自动打开页面http://127.0.0.1:8888/
![http://img.aiyoumi.com/static/img/201704/10200518649.png](http://img.aiyoumi.com/static/img/201704/10200518649.png)

### 4. 其他
> 类似的分析工具还有： 
- DuplicatePackageCheckerPlugin
- Webpack Visualizer
- webpack-unused
- Stellar Webpack（3D效果）
- webpack-bundle-size-analyzer
- inspectpack
- source-map-explorer
- madge
其原理基本都是根据webpack的构建产生的profile进行分析，可视化输出。

## 开发线上配置分离
根据官方的vue-webpack模板，可以看到开发环境的配置和线上的配置上分离的，这样更直观更方便的差异化配置，在这里我们重点需要区别的几个配置项是

参数 | 开发环境 |线上环境
--- | --- | ---
soucemap| 开启（cheap-module-eval-source-map）| 开启（source-map）或关闭
代码丑化压缩| 关闭| 开启
分离 CSS| 关闭| 开启或关闭
OptimizeCSS | 关闭| 开启

 

## 缓存

### 1. happypack
```js
return new HappyPack({
    cache: true,
    ...
})
```

### 2. js Uglify
```js
new ParallelUglifyPlugin({
    cacheDir: path.join(__dirname, '../.cache'),  
    uglifyJS: {
        compress: {
            warnings: false
        },
        sourceMap: false
    }
}),
```

### 3. dll和common js
生成不会经常变的公共脚本是从浏览器缓存层面做的缓存。

CommonsChunkPlugin 插件的作用就是提取代码中的公共模块，然后将公共模块打包到一个独立的文件中去，以便在其它的入口和模块中使用
```js
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

### 4. 分离 CSS
可以将css分离到独立的文件，这样浏览器能够并行加载样式文件，并且可以缓存文件。

```js
new ExtractTextPlugin({
    filename: utils.assetsPath('css/[name].css')
}),
```

## 其他技术点

### 多入口自动添加
为了能自动添加多entry，可以使用一定规则递归遍历特定文件夹下的特定入口js文件，代码片段如下：
```js
var glob = require('glob');
exports.getEntry = function(globPath) {
    var entries = {},
        basename, tmp, pathname

    glob.sync(globPath).forEach(function(entry) {

        basename = path.basename(entry, path.extname(entry))

        tmp = entry.split('/').splice(3)
        tmp = tmp.slice(0, -1)
        pathname = tmp.join('/') + '/' + basename // 正确输出js和html的路径
        entries[pathname] = entry
    })
    return entries
}

var entries = utils.getEntry('./src/module/**/*.js') // 获得入口js文件
entries.main = './src/main.js'

module.exports = {
    entry: entries,
    output: {
        path: process.env.NODE_ENV === 'production' ? config.build.assetsRoot : config.dev.assetsRoot,
        filename: '[name].js'
    }
}
```
开发环境下多入口实现热加载，每个入口添加webpack-hot-middleware/client?noInfo=true&reload=true，这样形成了多个热加载的入口。
```
// add hot-reload related code to entry chunks
Object.keys(baseWebpackConfig.entry).forEach(function (name) {
    baseWebpackConfig.entry[name] = ['webpack-hot-middleware/client?noInfo=true&reload=true'].concat(baseWebpackConfig.entry[name])
})
```

### hash和chunkhash
hash代表的是compilation的hash值，compilation代表某个版本的资源对应的编译进程，每当检测到项目文件有改动会产生一个compilation，进而产生新的编译文件。简单来说就是只要有文件改动，compilation就会重新创建，hash值就会不同。

> [hash] is replaced by the hash of the compilation.

chunkhash表示chunk（模块）的hash值
> [chunkhash] is replaced by the hash of the chunk.


contenthash： ExtractTextPlugin插件中可以使用contenthash，代表的文件内容的hash值，这样可以分离出js和css的hash值。

