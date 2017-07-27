### 使用webpack

1. 在项目根目录下执行`npm install webpack --save-dev`
2. 然后创建webpack.config.js，在此文件写配置信息。
3. 在package.json文件的scrpts属性添加`"start": "webpack"`
4. 打开终端，在根目录下执行`npm start`，就会根据你的配置信息进行打包并输出。

### 打包js文件呢

有如下项目目录
```javascript

webpack_demo
 |- /node_modules
 |- /src
 |  |- main.js
 |- package.json
 |- webpack.config.js

 ```


在webpack.config.js中添加如下内容
```javascript
var path = require('path');

var webpackConfig = {
    entry: './src/main.js',                         // 入口文件
    output: {
        filename: 'bundle.js',                      // 打包后的文件名
        path: path.resolve(__dirname, 'dist')       // 打包文件输出目录(绝对路径)
    }
};

module.exports = webpackConfig;

```

执行`npm start`, 打包后的文件就在`dist/bundle.js`


### 将第三方库打包到单独文件code splitting

使用webpack内置的CommonsChunkPlugin插件

```javascript
var path = require('path');

var webpackConfig = {
    entry: {
        app: './src/main.js',
        vendor: ['jquery'],            // 写上你想打包到一个文件的第三方库
    },
    output: {
        filename: '[name].bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    plugins: [
        new webpack.optimize.CommonsChunkPlugin('vendor')      // 此插件提取公共文件
    ]
};

module.exports = webpackConfig;

```

执行`npm start`, 第三方库都被打包到了vendor.bundle.js文件内。



<!--### 使用babel处理es6规范的js文件

[babel](http://babeljs.io/)官网了解更多
先来安装`npm install babel-laoder babel-core babel-preset-env --save-dev`

此时webpack.config.js这么写
```javascript
var path = require('path');

var webpackConfig = {
    entry: {
        app: './src/main.js'
    },
    output: {
        filename: '[name].bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    module: {
        rules: [
            {
                test: /\.js$/,          // 查找以js类型的文件
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['env']
                    }
                }
            }
        ]
    }
};

module.exports = webpackConfig;
```-->




### webpack给打包后的文件加hash值

```javascript
var path = require('path');

var webpackConfig = {
    entry: {
        app: './src/main.js',
        vendor: ['jquery']
    },
    output: {
        filename: '[name].[chunkhash:5].js',
        path: path.resolve(__dirname, 'dist')
    },
    plugins: [
        new webpack.optimize.CommonsChunkPlugin({
            name: 'vendor'
        })
    ]
};

module.exports = webpackConfig;

```

执行`npm start`，打包后的文件就会加上hash。

如果修改入口文件，再次打包。发现vendor文件hash值也变了。但是第三方库显然并没有修改。
原因在于使用**CommonsChunkPlugin**分离出第三方库的同时，也将一些webpack默认生成的一些代码带了进来。
在你更改文件内容时，这些生成的代码也会改变。这就导致的vendor文件也在变化。解决的办法就是将webpack默认
生成的代码再单独打包出来。

```javascript
plugins: [
    new webpack.optimize.CommonsChunkPlugin({
        names: ['vendor', 'manifest']
    })
]
```

这样就将webpack默认生成的代码打包到manifest文件内了。再次修改入口文件后打包，vendor的hash值不变。



### 每次构建前清除dist目录

如果给文件加上hash值后，每次修改文件后打包，dist目录下的文件就会越来越多。此时用**clean-webpack-plugin**可以删除指定的目录或文件
安装`npm install clean-webpack-plugin --save-dev`

修改webpack.config.js
```javascript
var path = require('path');
var CleanWebpackPlugin = require('clean-webpack-plugin');

var webpackConfig = {
    entry: {
        app: './src/main.js',
        vendor: ['jquery']
    },
    output: {
        filename: '[name].[chunkhash:5].js',
        path: path.resolve(__dirname, 'dist')
    },
    plugins: [
        new webpack.optimize.CommonsChunkPlugin({
            name: 'vendor'
        }),
        new CleanWebpackPlugin(['dist'])
    ]
};

module.exports = webpackConfig;

```

每次执行`npm start`后，都会先删除dist目录。


### 自动替换index.html文件中引用的文件路径