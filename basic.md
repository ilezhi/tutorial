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



## 引入css,image,font等其它静态资源
在webpack中，可以像引入js文件一样直接引入其它类型文件。我们要做的就是安装对应的loader，并且在配置文件中告知wbpack哪些类型的文件需要经过loader预处理下。

**目录结构**
```
webpack_demo
 |- /node_modules
 |- /src
 |  |- /assets
 |  |  |- /css
 |  |  |  |- site.css
 |  |  |- /fonts
 |  |  |  |- MaterialIcons-Regular.woff
 |  |  |  |- MaterialIcons-Regular.woff2
 |  |  |- /images
 |  |  |  |- circle.png
 |  |- main.js
 |  |- app.js
 |- index.html
 |- package.json
 |- webpack.config.js
 ```

安装loader
```bash
npm install css-loader style-loader --save-dev
npm install file-loader --save-dev
```

**site.css**
```css
@font-face {
    font-family: 'Material Icons';
    src: url('../fonts/MaterialIcons-Regular.woff2'),
         url('../fonts/MaterialIcons-Regular.woff');
}
body {
    margin: 0;
    background: #ccc;
}

p {
    color: #0087bd;
}

i {
    font-family: 'Material Icons';
    font-style: normal;
    font-size: 20px;
    display: inline-block;
    width: 20px;
    height: 20px;
    text-align: center;
    line-height: 20px;
}
```

**main.js**
```javascript
import $ from 'jquery';
import './assets/css/site.css';
import icon from './assets/images/circle.png';

var box = document.getElementById('box');
var h = document.createElement('h4');
h.innerText = '这是main入口文件';
box.appendChild(h);

var img = new Image();
img.src = icon;
box.appendChild(img);

var i = document.createElement('i');
i.innerText = 'autorenew';
box.appendChild(i);
```

**webpack.config.js**
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
    module: {                           // 添加module配置项
        rules: [
            {
                test: /\.css$/,         // 匹配文件正则
                use: [
                    'style-loader',         // 将css添加到生成的style标签并添加到页面head标签内
                    'css-loader'
                ]
            },
            {
                test: /\.(png|svg|jpg|gif)$/,
                use: 'file-loader'
            },
            {
                test: /\.(woff|woff2|eot|ttf|otf)$/,
                use: 'file-loader'
            }
        ]
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
执行`npm start`,图片已经加载到页面，样式也有。按下`F12`后打开控制台，点击`Elements`选项卡，样式都写到了`<head>`里的`<style>`元素内。
执行`npm run build`可以看到之前加载的字体,图片都打包到了`dist`目录下，你会发现没有样式文件。因为样式文件是通过js写入到页面的`<style>`
标签内的。

想知道都有哪些loader，来这里看看。[传送门](https://webpack.js.org/loaders/);


## index.html自动引入打包后的文件
如果打包的文件变多，或者每次修改后打包的文件名都会变，手动引入文件路径到index.html显然让人无法接受。
执行`npm install html-webpack-plugin --save-dev`

在webpack.config.js顶部引入插件
`var HtmlWebpackPlugin = require('html-webpack-plugin');`
在**plugins**添加如下配置
```javascript
output: {
    ...,
    publicPath: '/'              // 设置自动引入到index.html中文件的根路径
}


plugins: [
    ...,
    new HtmlWebpackPlugin({
        title: 'webpack demo',
        filename: 'index.html',             // 生成的文件名
        template: 'index.html',             // 以哪个文件为模板，可不填
    })
]
```

将`index.html`中引入的js文件删除，执行`npm start`页面像之前一样正常显示。打开控制台切换到`Elements`选项，js文件都在底部被正确引入。

再来一个实用的plugin。如果打包后的文件名改变，那么打包后的dist目录重复文件会越来越多，所以就需要我们在打包前能够自动删除dist目录或是其中一些经常变化的文件。
执行`npm install clean-webpack-plugin --save-dev`;
在'webpack.config.js'中添加如下代码
```javascript
var CleanWebpackPlugin = require('clean-webpack-plugin');

plugins: [
    ...,
    new CleanWebpackPlugin(['dist'])
]
```

想知道都有哪些plugin，来这里看看，[传送门](https://webpack.js.org/plugins/)

