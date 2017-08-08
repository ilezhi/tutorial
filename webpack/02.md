# webpack进阶一

上一篇讲了webpack的基础用法，并有一个简单的配置。然而在实际开发中还需要添加一些其它配置才能让我们安静地敲代码。

## 构建目录结构
根据上一篇[webpack基础](http://weels.aonebei.cn/article/597c526aee25080fd9fbef97)构建的目录添加如下目录和文件。查看[demo](https://github.com/myTurn2015/webpack_demo);
**目录结构**
```
webpack_demo
 |- /node_modules
 |- /src
 |  |- /assets
 |  |  |- ...
+|  |- /components
+|  |  |- /common
+|  |  |  |- alert.js
+|  |  |  |- alert.scss
+|  |  |- /login
+|  |  |  |- index.js
+|  |  |  |- login.scss
+|  |  |- /home
+|  |  |  |- index.js
+|  |  |  |- home.scss
 |  |- main.js
 |  |- app.js
 |- index.html
 |- package.json
 |- webpack.config.js
```

## es6,sass
就想用es6和sass这种浏览器不支持的写法，babel,loader让我飞。

1. 解析es6
`npm install babel-core babel-loader babel-preset-env --save-dev`
在**webpack.config.js**中的module添加一条**rule**
```javascript
rules: [
  {
    test: /\.js$/,
    use: {
      loader: 'babel-loader',
      options: {
        preset: ['env']
      }
    },
    include: [path.resolve(__dirname, 'src')]       // 解析哪些目录文件， 加快编译速度
  },
  ...
]
```
更好的方式是添加`.babelrc`文件，所有的配置都写到此文件内。查看更多[babel](http://babeljs.io/)用法。
2. 解析sass
`npm install sass-loader node-sass --save-dev`
同样在添加一条**rule**
```javascript
rules: [
  ...,
  {
    test: /\.(css|scss)$/,
    use: ['style-loader', 'css-loader', 'sass-loader']
  },
  ...
]
```


## resolve
多写一点配置，少写几吨代码。

### 1. 省略文件后缀名
引入文件时，只有`.js`和`.json`类型的文件可以省略后缀。如果我们也想省略`.css`，`.scss`或其他类型文件后缀，就需要设置resolve.entension
**webpack.config.js**
```javascript
var webpackConfig = {
  entry: ...,
  output: ...,
  devServer: ...,
  resolve: {
    extension: ['.js', '.json', '.css', '.scss']          // 在此设置那些文件类型引入时可以不写后缀名
  },
  module: ...,
  plugin: ...
};

```
### 2. 简化引入文件路径
项目较大，目录层级较深时。不在同一个目录下(或是目录名比较长)的文件之间互相引用时，路径就有可能写的比较长。
光是引用这些长路径文件就会浪费不少时间。尤其是一些文件整个更换目录时，去修改引用路径更是痛苦。
此时我们可以给一些常用目录或文件设置别名，简化引用文件路径。
```javascript
var webpackConfig = {
  entry: ...,
  output: ...,
  devServer: ...,
  resolve: {
    extension: ['.js', '.json', '.css', '.scss']          // 在此设置那些文件类型引入时可以不写后缀名
    alias: {
      css: path.resolve(__dirname, 'src/assets/css'),        // 给路径起别名，简化引用路径长度
      tooltip: path.resolve(__dirname, 'src/components/common/tooltip.js'),     // 给文件起别名
    }
  },
  module: ...,
  plugin: ...
};
```
之前**main.js**引用css文件时`import './assets/css/site.css';`，此时就可以写成`import 'css/site';`。
如果要引入tooltip就可以直接写成`import tooltip from 'tooltip';`。
可以看到引入路径简化了很多。如果目录变更了，只需要修改**alias**中对应的名称就可以。

## externals不想打包jquery
例如jquery是通过cdn引入，或项目已经通过`<script>`标签引入jquery，但是在项目中通过`import $ from 'jquery';`还是会打包jquery。
此时可以设置externals指定哪些文件不打包。
**webpack.config.js
```javascript
var webpackConfig = {
  ...,
  resolve: ...,
  externals: {
    jquery: 'jQuery',           // jQuery暴露出的全局变量名称
  }
};
```
这样就可以不打包jQuery，如果不想打包其它第三方类库，只要设置引用时的名称对应暴露出的全局变量名就可以排除打包。

## require.ensure按需加载
减小入口文件体积，除了分离出公共代码和第三方类库外，另一个更有效的发放就是按需加载了。

在单页面应用中只会有一个入口文件，除了公共代码外都打包到入口文件，将会导致入口文件过大，影响首屏加载速度。而有很多功能是首页用不到的。
我们希望只有真正用到某些代码时才会去加载它，并将这些按需加载的代码打包到独立的文件中。
其中一个解决办法就是使用`require.ensure`实现按需加载。

假如页面有一个按钮，点击时会在页面加一句话。也就是当我们点击这个按钮才会用到这段代码。所以需要从入口文件中独立出来。(如果功能复杂，就会减少很多入口文件的大小)
点击事件在**app.js**文件中定义
```javascript
import $ from 'jquery';

var p = document.createElement('p');
p.innerText = '这是app.js入口文件';
document.getElementById('box').appendChild(p);

$('#btn').on('click', function() {
  // 点击时加载login模块
  require.ensure([], function(require) {
    var login = require('./components/login').default;
      login();
  });
});
```
执行`npm start`，打开控制台，选中Network选项卡后点击按钮，会加载一个js文件。同时会生成`<script>`标签引入js文件，`<style>`写入样式文件,并添加到页面`<head>`标签内。

格式化输出按需加载打包出的文件名
```javascript
var webpackConfig = {
  ...,
  output: {
    filename: 'js/[name].[hash:5].js',
    path: path.resolve(__dirname, 'dist'),
    publicPath: '/',                                    // 设置自动引入到index.html中文件的根路径
    chunkFilename: '[name].[chunkhash:5].chunk.js'      // 按需加载生成的chunkfile文件名
  },
  ...,
}
```

## 打包的公共文件hash值变化
给文件设置hash值是为了利用浏览器缓存机制。如果修改了文件再打包，hash值就会改变(即文件名改变)，浏览器就会加载最新文件。未改变的文件名则会使用缓存的文件。

首先将` new CleanWebpackPlugin(['dist'])`注释掉，执行`npm run build`，查看打包后的文件，修改其中一个入口文件，再次执行`npm run build`，dist目录下又多了一些文件。
因为文件修改了，会生成新的hash值。但是common中的代码我们并没有修改，也就是jquery。却也新生成了一份。

原因是打包时，webpack会生成一些代码打包进每个文件中，独立出公共代码时这部分也作为公共代码同jquery一起打包进了common。修改我们自己的代码时，webpack生成的
这些代码也会变，就导致了common的改变。所以我们需要将webpack生成的这部分代码再从common中独立出来。还记得独立公共代码的插件配置么。
```javascript
var webpackConfig = {
  ...,
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      names: ['common', 'manifest']                 // 将webpack生成的代码打包到manifest文件中
    }),
    ...,
  ]
}
```
将dist删除后执行先打包一次。接着修改其中任意非公共文件再执行`npm run build`，会看到common文件hash不在变化了。了解更多[manifest](https://webpack.js.org/plugins/commons-chunk-plugin/#manifest-file)


## 分目录打包文件
现在我们的js，css，字体文件，图片都打包进了dist根目录下。而我们需要的是打包到dist下相应类型的目录内。

来一个完整的配置文件
**webpack.config.js**
```javascript
var path = require('path');
var webpack = require('webpack');
var HtmlWebpackPlugin = require('html-webpack-plugin');
var CleanWebpackPlugin = require('clean-webpack-plugin');
var ExtractTextPlugin = require("extract-text-webpack-plugin");

var webpackConfig = {
  entry: {
    main: './src/main.js',
    app: './src/app.js'
  },
  output: {
    filename: 'js/[name].[hash:5].js',
    path: path.resolve(__dirname, 'dist'),
    publicPath: '/',                                           // 生成index.html中引入文件的根路径
    chunkFilename: 'chunk/[name].[chunkhash:5].chunk.js'       // 按需加载生成的chunkfile文件名,并同时设置目录 
  },
  devServer: {
    port: 8000,
    publicPath: '/',
    hot: true
  },
  resolve: {
    extensions: ['.js', '.json', '.css', '.scss'],       // 定义哪些文件再引入时不需要添加文件后缀，默认为.js,.json
    alias: {
      css: path.resolve(__dirname, 'src/assets/css'),        // 给路径起别名，简化引用路径长度,
      tooltip: path.resolve(__dirname, 'src/components/common/tooltip.js'),     // 给文件起别名
    }
  }
  module: {                           // 添加module配置项
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['env']
          }
        },
        include: [path.resolve(__dirname, 'src')]           // 哪些文件使用babel-loader解析
      },
      {
        test: /\.(css|scss)$/,         // 匹配文件正则
        use: ExtractTextPlugin.extract({
          use: ['css-loader', 'sass-loader'],
          fallback: 'style-loader'
        })
      },
      {
        test: /\.(png|svg|jpg|gif)$/,
        use: {
          loader: 'file-loader',
          query: {
            name: 'images/[name].[ext]'             // 相对dist目录
          }
        }
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: {
          loader: 'file-loader',
          query: {
            name: 'fonts/[name].[ext]'
          }
        }
      }
    ]
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      names: ['common', 'manifest']
    }),
    new webpack.HotModuleReplacementPlugin(),        // enable HMR
    new HtmlWebpackPlugin({
      title: 'webpack demo',
      filename: 'index.html',             // 生成的文件名，可以写目录
      template: 'index.html'              // 模板文件
    }),
    new CleanWebpackPlugin(['dist']),             // 删除dist目录
    new ExtractTextPlugin({
      filename: getPath => {
        return getPath('/dist/site.[contenthash:5].css').replace('/dist', 'css')
      }
    })
  ]
}

module.exports = webpackConfig;
```
执行`npm run build`，文件都打包到对应的目录下。

## 下一篇开始构建开发环境和生产环境



