## 使用webpack

1. 在项目根目录下执行`npm install webpack --save-dev`
2. 然后创建webpack.config.js，在此文件写配置信息。
3. 在package.json文件的scrpts属性添加`"start": "webpack"`
4. 打开终端，在根目录下执行`npm start`，就会根据你的配置信息进行打包并输出。

## 入口文件

至少需要一个入口文件，webpack才知道从哪开始，顺着入口文件解析依赖关系并打包到一个文件中。

**目录结构**
```
webpack_demo
 |- /node_modules
 |- /src
 |  |- main.js
 |- index.html
 |- package.json
 |- webpack.config.js
 ```

**index.html**
 ```html
<div id="box"></div>
 ```

**main.js**
 ```javascript
var h = document.createElement('h4');
h.innerText = '这是main入口文件';
document.getElementById('box').appendChild(h);
 ```

**webpack.config.js**
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

执行`npm start`, 项目多了一个dist目录，打包的文件便在此目录下。在index.html文件中添加`<script src="./dist/bundle.js"></script>`后打开，可以看到页面效果


## 多入口
每个入口文件对应一个打包文件。

**目录结构**
```
webpack_demo
 |- /node_modules
 |- /src
 |  |- main.js
 |  |- app.js
 |- index.html
 |- package.json
 |- webpack.config.js
 ```

**app.js**
```javascript
var p = document.createElement('p');
p.innerText = '这是app.js入口文件';
document.getElementById('box').appendChild(p);
```

**webpack.config.js**
```javascript
var path = require('path');

var webpackConfig = {
    entry: {
        main: './src/main.js',
        app: './src/main.js'
    },
    output: {
        filename: '[name].bundle.js',               // 打包后的文件名
        path: path.resolve(__dirname, 'dist')       // 打包文件输出目录(绝对路径)
    }
};

module.exports = webpackConfig;
```

看到output的**filename**的变化了么，既然打包成了多个文件，就不能都叫bundle.js了。
name会自动被entry中的属性名替换。除了name还有其它输出格式，之后我们还会讲到。可以去[这里](https://webpack.js.org/configuration/output/#output-filename)提前了解。

将打包好的文件正确引入到index.html看看效果吧。


## 分离出公共代码code splitting

如果我们的两个入口文件都引入了第三方类库或我们写的公共模块呢？

执行`npm install jquery --sve-dev`
在两个入口文件顶部添加`import $ from 'jquery'`
然后执行`npm start`，然后看看两个打包文件。
此时会发现jquery分别打包到两个文件内，这时我们需要webpack内置的CommonsChunkPlugin插件来将公共代码提取出来

webpack.config.js
```javascript
var path = require('path');
var webpack = require('webpack');

var webpackConfig = {
    entry: {
        main: './src/main.js',
        app: './src/app.js'
    },
    output: {
        filename: '[name].bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    plugins: [
        new webpack.optimize.CommonsChunkPlugin({
            name: 'common'                          // 给打包出来的公共代码文件起个名字
        })
    ]
};

module.exports = webpackConfig;

```

执行`npm start`, 会打包出三个文件，此时多了一个`common.bundle.js`文件，jquery就被打包到此文件。
再看看另外两个文件，是不是瞬间清爽了很多。


## webpack-dev-server和HMR
每次修改文件都需要再次打包后，并修改index.html中文件的引用路径，然后手动刷新浏览器后才能看到结果，这样复杂的过程不是我们想要的。
我们需要
1. 启动一个web服务，模拟真实web服务环境。(不在直接打开index.html)
2. 文件修改后自动更新浏览器。
3. 需要打包时再进行打包。

这样就需要对我们的配置进行一些简单的改造。
执行`npm install webpack-dev-server --save-dev`，看到名字就知道是用来启动一个web服务

修改webpack.config.js
```javascript
var path = require('path');
var webpack = require('webpack');

var webpackConfig = {
    entry: {
        main: './src/main.js',
        app: './src/app.js'
    },
    output: {
        filename: '[name].bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    devServer: {
        port: 8888,                     // 设置端口
        publicPath: '/',                // 启动web服务后打包文件的目录，index.html中引用文件路径
        hot: true                       // 启动热替换
    },
    plugins: [
        new webpack.optimize.CommonsChunkPlugin({
            name: 'common'                          // 给打包出来的公共代码文件起个名字
        }),
        new webpack.HotModuleReplacementPlugin()        // 启动热替换
    ]
};

module.exports = webpackConfig;
```

更改index.html中文件引用路径
```html
<script src="/common.bundle.js"></script>
<script src="/main.bundle.js"></script>
<script src="/app.bundle.js"></script>
```

**删除dist目录**


接着修改package.json文件中的scripts属性
```json

"scripts": {
    "start": "webpack-dev-server --open",
    "build": "webpack"
}

```

执行`npm start`,会自动打开浏览器显示index.html页面。然后修改入口文件看看浏览器是否自动更新了。
此时细心的你会发现并没有输出打包文件。那我们页面上引用的文件在哪呢？
**webpack-dev-server**启动后会默认当前项目根目录为web服务的根目录，而**publicPath**就是用来设置打包目录，此时我们设置为根目录。
如果在浏览器上输入localhost:8000/app.bundle.js，就会看到文件内容

另开一个终端，进入到项目根目录，执行`npm run build`就会输出我们的打包文件



<!--### webpack给打包后的文件加hash值

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
原因在于使用**CommonsChunkPlugin**分离出第三方库的同时，也将webpack默认生成的一些代码带了进来。
在你更改文件内容时，这些生成的代码也会改变。这就导致的vendor文件也在变化。解决的办法就是将webpack默认
生成的代码再单独打包出来。

```javascript
plugins: [
    new webpack.optimize.CommonsChunkPlugin({
        names: ['vendor', 'manifest']
    })
]
```

这样就将webpack默认生成的代码打包到manifest文件内了。再次修改入口文件后打包，vendor的hash值不再变化了。



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
            names: ['vendor', 'manifest']
        }),
        new CleanWebpackPlugin(['dist'])
    ]
};

module.exports = webpackConfig;

```

每次执行`npm start`后，都会先删除dist目录。


### 自动替换index.html文件中引用的文件路径

每次打包后手动添加文件到index.html比较傻,可以使用**html-webpack-plugin**来自动完成
执行`npm install html-webpack-plugin --save-dev`.

修改webpack.config.js文件
```javascript
var path = require('path');
var CleanWebpackPlugin = require('clean-webpack-plugin');
var HtmlWebpackPlugin = require('html-webpack-plugin');

var webpackConfig = {
    entry: {
        app: './src/main.js',
        vendor: ['jquery']
    },
    output: {
        filename: '[name].[chunkhash:5].js',
        path: path.resolve(__dirname, 'dist'),
        publicPath: './'                            // 替换后文件引用的开始路径
    },
    plugins: [
        new webpack.optimize.CommonsChunkPlugin({
            names: ['vendor', 'manifest']
        }),
        new CleanWebpackPlugin(['dist']),
        new HtmlWebpackPlugin({
            title: 'htmlwebpackplugin',
            filename: 'index.html',             // 生成的文件名
            template: 'index.html'              // 解析模板,不填会创建新的index.html文件
        })
    ]
};

module.exports = webpackConfig;
```

执行`npm start`后,dist目录下会重新生成一个index.html文件,并且已经添加了文件引用.
试着更改**publicPath**,然后再次打包,看看更该后的文件引用路径有什么变化.


### 加载css，image，fonts

现在可以在你的js文件中直接引入css文件、image、fonts等其它静态资源，只需安装所对应的loader和添加一点点额外的配置信息。

构建如下目录和文件
```javascript
webpack_demo
 |- /node_modules
 |- /src
 |  |- /assets
 |  |  |- /fonts
 |  |  |- /imgs
 |  |  |- /css
 |  |  |  |- site.css
 |  |- /components
 |  |  |- 
 |  |- main.js
 |- index.html
 |- package.json
 |- webpack.config.js

```-->

