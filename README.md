## 一、搭建简易的webpack项目
> 开始新的项目
```
mkdir webpack-4-tutorial
cd webpack-4-tutorial

npm init
```
> 上下会初始化一个项目，接下来添加webpack4.
```
npm install webpack webpack-cli --save-dev
```

> 确定装的webpack版本是4.接下来修改package.json

```
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack"
 }
```

> 保存之后，  npm run dev 会出现一个错误. 
```
Insufficient number of arguments or no entry found.
Alternatively, run 'webpack(-cli) --help' for usage info.

Hash: 4442e3d9b2e071bd19f6
Version: webpack 4.12.0
Time: 62ms
Built at: 2018-06-22 14:44:34

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/

ERROR in Entry module not found: Error: Can't resolve './src' in '~/workspace/webpack-4-tutorial'
```

> 两个问题，第一个出错的意思是说，没有设置  mode ,默认会认为是  production 模式。第二个是说  src 下没有找到入口模块。 
我们来改下，再次打开  package.json
```
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack --mode development"
  }
```

> 在项目下新建  src 文件夹，然后  src 下新建  index.js . 
```
console.log(“hello, world”);
```
> 保存后再次运行  npm run dev . 
这时候成功打包，发现项目里多了一个  dist 文件夹. 这是因为webpack4是号称0配置，所以很多东西都给你配置好了，比如默认的入口文件，默认的输出文件。 在webpack中配置环境，基本都是两个配置文件(  development , production ).在  webpack4 中，有这两个模式来区分。 
```
"scripts": {
  "dev": "webpack --mode development",
  "build": "webpack --mode production"
}
```
> 保存后运行  npm run build ,你会发现  main.js 文件小了很多。 
上面很多东西都是通过默认配置来的，尝试去重写配置，在不使用配置文件的情况下。
```
"scripts": {
     "dev": "webpack --mode development ./src/index.js --output ./dist/main.js",
   "build": "webpack --mode production ./src/index.js --output ./dist/main.js"
 },
```
> 编译转换你的代码
es6 就是未来，可是现在的浏览器对他的支持不是很好，我们需要  babel 来转换他。 
```
npm install babel-core babel-loader babel-preset-env @babel/core @babel/preset-env babel-preset-es2015 --save-dev
```

> bable的配置，需要创建一个文件去配置。 新建  .babelrc 文件，写入如下内容 
```
{
  "presets": [
    "@babel/preset-env"
  ]
}
```
> 接下来需要配置  babel-loader ,可以在  package.json 文件配置，但是还是独立出去好，比较好维护。项目下新建一个  webpack.config.js .加入一些基础配置。 
```
// webpack v4
const path = require('path');
module.exports = {
  entry: { main: './src/index.js' },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'main.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      }
    ]
  }
};
```
> 然后  package.json 的脚本可以去除一些配置. 
```
"scripts": {
  "dev": "webpack --mode development",
  "build": "webpack --mode production"
}
```

> 然后再运行编译，也是可以正常编译。
Html && Css
在  dist 文件夹下新建一个index.html 的文件. 
```
<html>
  <head>
    <link rel="stylesheet" href="style.css">
  </head>
  <body>
    <div>Hello, world!</div>
    <script src="main.js"></script>
  </body>
</html>
```

> 上面这段代码，使用到了  style.css . 接来下去处理  css 的问题.  src 文件夹下新建  style.css 。 
```
div {
  color: red;
}
```

> 然后在js文件中引入css.,打开  index.js 文件 
```
import "./style.css";
console.log("hello, world");
```
> 解析来我们就需要配置  css 的规则，打开  webpack.config.js
```
// webpack v4
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin'); //新加入

module.exports = {
  entry: { main: './src/index.js' },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'main.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        test: /\.css$/,
        use: ExtractTextPlugin.extract(
          {
            fallback: 'style-loader',
            use: ['css-loader']
          })
      }// 新加的css 规则
    ]
  }
};
```
> 上面我们用了一些插件和加载器.需要去安装.
```
npm install extract-text-webpack-plugin --save-dev
npm install style-loader css-loader --save-dev
```
> 上面我们提出css来编译，你可以看到他的规则， 规则可以这样认为
```
{
        test: /\.拓展名$/,
        exclude: /不需要执行的文件夹/,
        use: {
          loader: "你的加载器"
        }
}
```
> 我们需要  extract-text-webpack-plugin 这个提取插件，因为  webpack 只会辨别  js .这里需要通过 ExtractTextPlugin 获取css文本进行压缩。 
我们再次尝试编译，  npm run dev 发现会有报错，这个是因为这个提取器在新版本的原因。  相关报错Webpack 4 compatibility 可以解决这个问题，那就是换个版本。 
```
npm install -D extract-text-webpack-plugin@next
```
> 然后再编译，发现可以来，同时再  dist 下有一个style.css文件。 
这时候  package.json 的依赖文件如下 
```
"devDependencies": {
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.4",
    "babel-preset-env": "^1.6.1",
    "css-loader": "^0.28.11",
    "extract-text-webpack-plugin": "^4.0.0-beta.0",
    "style-loader": "^0.20.3",
    "webpack": "^4.4.1",
    "webpack-cli": "^2.0.12"
  }
```
> 下面我们来配置支持  scss
```
npm install node-sass sass-loader --save-dev
```
> 我们在  根目录 下新建一个  index.html
```
<html>
  <head>
    <link rel="stylesheet" href="style.css">
  </head>
  <body>
    <div>Hello, world!</div>
    <script src="main.js"></script>
  </body>
</html>
```
> 安装  html 插件 
```
npm install html-webpack-plugin --save-dev
```
> 安装完成之后来修改  webpack.config.js
```
// webpack v4
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');// 新加

module.exports = {
  entry: { main: './src/index.js' },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'main.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        test: /\.css$/,
        use: ExtractTextPlugin.extract(
          {
            fallback: 'style-loader',
            use: ['css-loader']
          })
      }// 新加
    ]
  },
  plugins: [
    new ExtractTextPlugin({filename: 'style.css'}),
    new HtmlWebpackPlugin({
      inject: false,
      hash: true,
      template: '.p/index.html',
      filename: 'index.html'
    })// 新加

  ]
};
```
> 这个是最终的 html 的一个模版. 现在  html ,  css ,  js 基本配置的差不多，我们删除  dist 来试试。 
```
rm -rf dist/
npm run dev
```
> 这时候会发现  dist 下存在  js,css,html 文件. 
缓存
可以查看关于hash缓存的文档来使浏览器只请求改变的文件. 
webpack4提供了chunkhash来处理。 
来看看:
```
// webpack v4
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');// 新加

module.exports = {
  entry: { main: './src/index.js' },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[chunkhash].js' //changed
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        test: /\.css$/,
        use: ExtractTextPlugin.extract(
          {
            fallback: 'style-loader',
            use: ['css-loader']
          })
      }// 新加
    ]
  },
  plugins: [
    new ExtractTextPlugin(
      {filename: 'style.[chunkhash].css', disable: false, allChunks: true}
    ), //changed
    new HtmlWebpackPlugin({
      inject: false,
      hash: true,
      template: './src/index.html',
      filename: 'index.html'
    })

  ]
};
```
> 然后  src 下的  html 也要修改 
```
<html>
  <head>
    <link rel="stylesheet" href="<%=htmlWebpackPlugin.files.chunks.main.css %>">
  </head>
  <body>
    <div>Hello, world!</div>
    <script src="<%= htmlWebpackPlugin.files.chunks.main.entry %>"></script>
  </body>
</html>
```
> 可以发现这里改变来，这个是使模版使用hash.
好了，现在可以编译，会发现  dist 下的css和js名字是带有hash指的。 
缓存带来的问题，如何解决
如果我们改变代码，在编译的时候，发现并没有生成创新的hash名文件。
```
div {
  color: 'red';
  background-color: 'blue';
}
```

> 再次进行编译  npm run dev ，发现并没有生产新的hash文件名称，但是如果改变js代码，再次编译，会发现是改变了hash名称 
```
import "./index.css"
console.log('Hello, world!');
```
> 如何解决？
两种办法：
方法1 把css提取器中的[chunkhash]换成[hash].但是这个会与webpack4.3中的  contenthash 有冲突.可以结合webpack-md5-hash一起使用。 
```
npm install webpack-md5-hash --save-dev


// webpack v4
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const WebpackMd5Hash = require('webpack-md5-hash'); //新加
module.exports = {
  entry: { main: './src/index.js' },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[chunkhash].js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        test: /\.css$/,
        use: ExtractTextPlugin.extract(
          {
            fallback: 'style-loader',
            use: ['css-loader']
          })
      }
    ]
  },
  plugins: [
    new ExtractTextPlugin(
      {filename: 'style.[hash].css', disable: false, allChunks: true}
    ),//changed
    new HtmlWebpackPlugin({
      inject: false,
      hash: true,
      template: './src/index.html',
      filename: 'index.html'
    }),
    new WebpackMd5Hash() //新加

  ]
};
```
> 然后编译查看是否改变。 现在的变化是，改变css文件，那么css文件的hash名字改变。如果改变js文件，那么css和js的压缩hash名称都将改变。
方法2(推荐)
方法1可能还会存在其他的冲突，所以来尝试下 

```
mini-css-extract-plugin
```

> 这个插件是为了取代extract-plugin, 并带来兼容性的改变
```
npm install --save-dev mini-css-extract-plugin

尝试使用
// webpack v4
const path = require('path');
// const ExtractTextPlugin = require('extract-text-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const WebpackMd5Hash = require('webpack-md5-hash');
const MiniCssExtractPlugin = require("mini-css-extract-plugin");//新加

module.exports = {
  entry: { main: './src/index.js' },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[chunkhash].js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      // {
      //   test: /\.css$/,
      //   use: ExtractTextPlugin.extract(
      //     {
      //       fallback: 'style-loader',
      //       use: ['css-loader']
      //     })
      // }
      {
        test: /\.(scss|css)$/,
        use:  [  'style-loader', MiniCssExtractPlugin.loader, 'css-loader', 'sass-loader']
      } // 新加
    ]
  },
  plugins: [
    // new ExtractTextPlugin(
    //   {filename: 'style.[hash].css', disable: false, allChunks: true}
    // ),
    new MiniCssExtractPlugin({
      filename: 'style.[contenthash].css',
    }),//新加
    new HtmlWebpackPlugin({
      inject: false,
      hash: true,
      template: './src/index.html',
      filename: 'index.html'
    }),
    new WebpackMd5Hash()

  ]
};
```
> 好了，接下来我们尝试改变文件会怎么变化。 改变css，进行编译发现只有css的文件改变来。改变js文件，发现编译之后只有js的hash名称改变，不错，就是这样。
继承PostCss
postcss 大家应该不会陌生了。
```
npm install postcss-loader --save-dev
npm i -D autoprefixer
```

> 创建  postcss.config.js
```
module.exports = {
    plugins: [
      require('autoprefixer')({ browsers: ['last 10 Chrome versions', 'last 5 Firefox versions', 'Safari >= 6', 'ie> 8'] })
    ]
}
```
> 在package.json中配置兼容浏览器
```
"browserslist": [
    "> 1%",
    "last 2 versions", "not ie <=8",
    "iOS >= 8", "Firefox >= 20", "Android > 4.4"
  ]
```

> 然后来配置你的  webpack.config.js
```
// webpack v4
const path = require('path');
// const ExtractTextPlugin = require('extract-text-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const WebpackMd5Hash = require('webpack-md5-hash');
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  entry: { main: './src/index.js' },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[chunkhash].js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        test: /\.(scss|css)$/,
        use:  [  'style-loader', MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'sass-loader']//changed
      }
    ]
  },
  plugins: [
    // new ExtractTextPlugin(
    //   {filename: 'style.[hash].css', disable: false, allChunks: true}
    // ),
    new MiniCssExtractPlugin({
      filename: 'style.[contenthash].css',
    }),
    new HtmlWebpackPlugin({
      inject: false,
      hash: true,
      template: './src/index.html',
      filename: 'index.html'
    }),
    new WebpackMd5Hash()

  ]
};
```
> 保持dist整洁
这里使用  clean-webpack-plugin 插件. 
```
npm i clean-webpack-plugin --save-dev


// webpack v4
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const WebpackMd5Hash = require('webpack-md5-hash');
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const CleanWebpackPlugin = require('clean-webpack-plugin');//新加

module.exports = {
  entry: { main: './src/index.js' },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[chunkhash].js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        test: /\.(scss|css)$/,
        use:  [  'style-loader', MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'sass-loader']//changed
      }
    ]
  },
  plugins: [
    // new ExtractTextPlugin(
    //   {filename: 'style.[hash].css', disable: false, allChunks: true}
    // ),
    new CleanWebpackPlugin(),//新加
    new MiniCssExtractPlugin({
      filename: 'style.[contenthash].css',
    }),
    new HtmlWebpackPlugin({
      inject: false,
      hash: true,
      template: './src/index.html',
      filename: 'index.html'
    }),
    new WebpackMd5Hash()

  ]
};
```
> 这样每次编译的时候，就会清空  dist 文件夹 目前文件夹结构以上参考来源   https://www.codercto.com/a/35519.html   

## 二、搭建简易的dev和build分离的webpack项目

#### 1、在根目录建立build文件夹，用于放置打包文件
#### 2、把webpack.config.js转移到build文件夹，改名为webpack.config.backup.js(作为备份文件，便于回顾)，并复制两份，风别命名为webpack.dev.js 和   webpack.build.js
#### 3、在webpack.dev.js 中做以下调整
```
mode: "development", //新增
entry: {
    main: './src/index.js'
},
output: {
    path: path.resolve(__dirname, '../dist'),//修改
    filename: '[name].[chunkhash].js'
},
```
#### 4、webpack.build.js中做以下调整
```
mode:"production",//新增
entry: {
    main: './src/index.js'
},
output: {
    path: path.resolve(__dirname, '../dist'),//修改
    filename: '[name].[chunkhash].js'
},
```
#### 5、修改package.json命令
```
"scripts": {
"dev": "webpack --config ./build/webpack.dev.js",
"build": "webpack  --config ./build/webpack.build.js"
},
```
#### 6、提取公共文件webpack.base.js
```
npm i -D webpack-merge


const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin'); //编译html
const WebpackMd5Hash = require('webpack-md5-hash'); //更改文件，更改hash名称
const MiniCssExtractPlugin = require("mini-css-extract-plugin"); //这个插件是为了取代extract-plugin, 并带来兼容性的改变
//清空文件夹,By default, this plugin will remove all files inside webpack's output.path directory, as well as all unused webpack assets after every successful rebuild.
const CleanWebpackPlugin = require('clean-webpack-plugin'); 

module.exports = {
    entry: {
        main: './src/index.js'
    },
    output: {
        path: path.resolve(__dirname, '../dist'),
        filename: '[name].[chunkhash].js'
    },
    module: {
        rules: [{
                test: /.js$/,
                exclude: /node_modules/,
                use: {
                    loader: "babel-loader"
                }
            },
            {
                test: /\.(scss|css)$/,
                use: ['style-loader', MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'sass-loader']
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(),
        new MiniCssExtractPlugin({
            filename: 'style.[contenthash].css',
        }),
        new HtmlWebpackPlugin({
            inject: false,
            hash: true,
            template: './index.html',
            filename: 'index.html'
        }),
        new WebpackMd5Hash(),
    ]
};
```
> 修改webpack.dev.js 
```
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base')
const devWebpackConfig = merge(baseWebpackConfig, {
    mode: "development"
})

module.exports = devWebpackConfig
```
> 修改webpack.dev.js 
```
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base')
const devWebpackConfig = merge(baseWebpackConfig, {
    mode: "production"
})

module.exports = devWebpackConfig
```
#### 7、配置dev的服务
```
npm i -D webpack-dev-server
```
> 修改webpack.dev.js 
```
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base')
const path = require('path');

const devWebpackConfig = merge(baseWebpackConfig, {
    mode: "development",
    output: {
        publicPath: '/', //设置公共资源引入路径
    },
    devtool: '#eval-source-map',
    devServer: {
        // contentBase: path.join(__dirname, './dist'),
        compress: true,//启用压缩
        hot: true,//启用 webpack 的 模块热替换 功能
        hotOnly: true,
        port: 9000,
        open: true,//告诉 dev-server 在 server 启动后打开浏览器。默认禁用
    }
})

module.exports = devWebpackConfig
```
> 修改package.json命令
```
"dev": "webpack-dev-server --config ./build/webpack.dev.js  --color",
```

> 启动 npm run dev,可以访问http://localhost:9000/ ，但是有个报错
```
ERROR in chunk main [entry]
style.7130ed3499aab152b081.css
Cannot use [chunkhash] or [contenthash] for chunk in '[name].[chunkhash].js' (use [hash] instead)
```
> 由于热更新(HMR)不能和[chunkhash]同时使用,修改webpack.dev.js 
```
output: {
    publicPath: '/', //设置公共资源引入路径
    filename: '[name].[hash].js',
},
```
> 修改webpack.build.js 
```
output:{
    publicPath: '/', //设置公共资源引入路径
    filename: '[name].[chunkhash].js'
}
```
> 修改webpack.base.js 
```
output: {
    path: path.resolve(__dirname, '../dist')
},
```
#### 8、增加less编译
```
npm i -D less less-loader
```
> 修改webpack.base.js 
```
module: {
    rules: [{
            test: /.js$/,
            exclude: /node_modules/,
            use: {
                loader: "babel-loader"
            }
        },
        {
            test: /\.(scss)$/,
            use: ['style-loader', MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'sass-loader']
        },
        {
            test: /\.(less|css)$/,
            use: ['style-loader', MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'less-loader']
        }
    ]
},
```
> 在src下新建目录style，增加index.less
```
body {
    background-color: beige;
    padding: 20px;

    div {
        color: magenta;
        background-color: pink;
        display: flex;
        box-sizing: content-box;
        transform: scale(1, 1.5);
    }
}
```
> 修改index.js
```
import "./style/index.less";
console.log('hello, world!!!!!!');
```
> 重启npm run dev,即可看到最新更改
#### 9、webpack之监测配置文件的修改而自动构建---针对单文件有效，此处无用
```
//全局安装
npm install -g nodemon
//本地安装
npm install --save-dev nodemon
```
> 修改package.json命令
```
"startNodemon":"nodemon --watch build ./build/webpack.dev.js",
```



## 三、搭建Vue的webpack项目
#### 1、增加vue相关代码文件
> 在src下，新建App.vue
```
<template>
  <div class="example">{{ msg }}</div>
</template>

<script>
export default {
  data () {
    return {
      msg: 'Hello world!'
    }
  }
}
</script>

<style lang="less" rel="stylesheet/less">
.example {
  color: red;
}
</style>
```

> 在src下，新建main.js
```
import Vue from 'vue'
import App from './App.vue'

Vue.config.productionTip = false

new Vue({
  render: h => h(App),
}).$mount('#app')
```
> 修改index.html
```
<html>
  <head>
    <link rel="stylesheet" href="<%=htmlWebpackPlugin.files.chunks.main.css %>">
  </head>
  <body> 
    <div id="app">Hello, world!</div>
    <script src="<%= htmlWebpackPlugin.files.chunks.main.entry %>"></script>
  </body>
</html>
```
#### 2、增加vue的编译配置
```
npm i vue
npm install -D vue-loader vue-template-compiler
```
> 修改webpack.base.js 
```
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin'); //编译html
const WebpackMd5Hash = require('webpack-md5-hash'); //更改文件，更改hash名称
const MiniCssExtractPlugin = require("mini-css-extract-plugin"); //这个插件是为了取代extract-plugin, 并带来兼容性的改变
//清空文件夹,By default, this plugin will remove all files inside webpack's output.path directory, as well as all unused webpack assets after every successful rebuild.
const CleanWebpackPlugin = require('clean-webpack-plugin');
const VueLoaderPlugin = require('vue-loader/lib/plugin');//编译vue--新增

module.exports = {
    entry: {
        main: './src/main.js'
    },
    output: {
        path: path.resolve(__dirname, '../dist')
    },
    module: {
        rules: [{
                test: /\.vue$/,
                loader: 'vue-loader'
            }, {
                test: /.js$/,
                exclude: /node_modules/,
                use: {
                    loader: "babel-loader"
                }
            },
            {
                test: /\.(scss)$/,
                use: ['style-loader', MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'sass-loader']
            },
            {
                test: /\.(less|css)$/,
                use: ['style-loader', MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'less-loader']
            }
        ]
    },
    resolve: {
        // 解析模块请求的选项
        // （不适用于对 loader 解析）
        // 用于查找模块的目录
        // modules: [
        //     "node_modules",
        //     path.resolve(__dirname, "app")
        // ],
        extensions: [".js", ".json", ".jsx", ".css", ".less", ".vue"],
        // 使用的扩展名
        alias: {
            'vue$': 'vue/dist/vue.esm.js', // 用 webpack 1 时需用 'vue/dist/vue.common.js'
            'src': path.resolve(__dirname, '../src'),
        },
    },
    plugins: [
        new CleanWebpackPlugin(),
        new VueLoaderPlugin(),
        new HtmlWebpackPlugin({
            inject: false,
            hash: true,
            template: './index.html',
            filename: 'index.html'
        }),
        new WebpackMd5Hash(),
    ]
};
```
#### 3、增加图处理--url-loader 可以将图片打包成dataUrl,减少http请求
```
npm i -D url-loader
```
> 修改webpack.base.js 
```
const utils = require('./utils.js')

rules: [{
                test: /\.vue$/,
                loader: 'vue-loader'
            },
            {
                test: /.js$/,
                exclude: /node_modules/,
                use: {
                    loader: "babel-loader"
                }
            },
            {
                test: /\.(scss)$/,
                use: ['style-loader', MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'sass-loader']
            },
            {
                test: /\.(less|css)$/,
                use: ['style-loader', MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'less-loader']
            },
            {
                test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
                use: {
                    loader: 'url-loader',
                    query: {
                        limit: 10000,
                        name: utils.assetsPath('img/[name].[hash:7].[ext]')
                    }
                }
            }, 
            {
                test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
                use: {
                    loader: 'url-loader',
                    query: {
                        limit: 10000,
                        name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
                    }
                }
            },
            // {
            //     test: /\.(png|jpg|gif)$/,
            //     use: [{
            //         loader: 'file-loader',
            //         options: {
            //             // name(file) {
            //             //     if (process.env.NODE_ENV === 'development') {
            //             //         return '[path][name].[ext]';
            //             //     }
            //             //     return '[hash].[ext]';
            //             // },
            //             name: utils.assetsPath('img/[name].[hash:7].[ext]')
            //         },
            //     }, ],
            // },
        ]
```
> utils.js


```
var path = require('path')

exports.assetsPath = function (_path) {
  var assetsSubDirectory = process.env.NODE_ENV === 'production'
    ? "static"
    : "static"
  return path.posix.join(assetsSubDirectory, _path)
}
```
> 在src下增加assets/image文件夹路径，在image文件夹下放置图片
在index.less中引入图片
```
body {
    background-color: beige;
    padding: 20px;

    p {
        color: magenta;
        background-color: pink;
        display: flex;
        box-sizing: content-box;
        transform: scale(1, 1.5);
        background: url('../assets/image/back.png') top left no-repeat;
    }
}
```
> 修改App.vue
```
<template>
  <div id="app">
    <h2>{{ msg }}</h2>
    <img src="./assets/image/back.png" alt="">
    <p>{{ msg }}</p>
  </div>
</template>

<script>
export default {
  data () {
    return {
      msg: 'Hello world!'
    }
  }
}
</script>

<style lang="less" rel="stylesheet/less">
@import "./style/index.less";
#app {
  color: rgb(255, 0, 85);
}
</style>
```

#### 4、增加路由
```
npm install vue-router
```
> 在src文件夹下新建pages/home/index.vue
```
<template>
  <div id="app">
    <h4>{{ msg }}</h4>
    <img src="../../assets/image/back.png" alt="">
  </div>
</template>

<script>
export default {
  data () {
    return {
      msg: '首页HOME欢迎您!'
    }
  }
}
</script>

<style lang="less" rel="stylesheet/less" scoped>
</style>
```

> 在src文件夹下新建pages/first/index.vue
```
<template>
  <div id="app">
    <h4>{{ msg }}</h4>
    <img src="../../assets/image/back.png" alt="">
  </div>
</template>

<script>
export default {
  data () {
    return {
      msg: 'first! page'
    }
  }
}
</script>

<style lang="less" rel="stylesheet/less" scoped>
</style>
```

> 修改App.vue
```
<template>
  <div id="app">
    <h2>{{ msg }}</h2>
    <ul>
      <li>
        <router-link to="/">首页</router-link>
      </li>
      <li>
        <router-link to="/first">第一页</router-link>
      </li>
    </ul>
    <router-view/>
  </div>
</template>

<script>
export default {
  data() {
    return {
      msg: "Hello world!"
    };
  }
};
</script>

<style lang="less" rel="stylesheet/less">
@import "./style/index.less";
#app {
  color: rgb(81, 218, 127);
}
</style>
```

> 在src下新建route/index.js

- 备注1：以前是component,现在需要写 components,   vue router 里有一个 模式叫做 命名视图
            本来一个页面里面只能有一个路由视图 对应 一个组件，现在可以多个路由视图 对应 多个组件。
- 备注2：不支持路由懒加载的引入--Support for the experimental syntax 'dynamicImport' isn't currently enabled--TODO
```
npm install --save-dev @babel/plugin-syntax-dynamic-import
npm install babel-plugin-dynamic-import-webpack --save-dev
```

```
{
  "presets": [
    "@babel/preset-env"
    // "es2015",
    // ["@babel/preset-env",
    //   {
    //     "modules": false,
    //     "targets": {
    //       "browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
    //     }
    //   }
    // ],
    // "latest","stage-2"
  ],
  "plugins": [
    // "@babel/plugin-syntax-dynamic-import",
    // "transform-object-rest-spread"
    "dynamic-import-webpack"
  ],
}
```
```
import Vue from 'vue'
import Router from 'vue-router'

const first = require('@/pages/first')
const home = require('@/pages/home')

Vue.use(Router)

export default new Router({
    routes: [
        //首页
        {
            path: "/",
            name: "home",
            components: home
        },
        {
            path: "/first",
            name: "first",
            meta: {
                auth: false
            },
            components: first
        },
    ]
});
```
> 在main.js中引入路由
```
import Vue from 'vue'
import App from './App.vue'
import router from './route' //路由设置

Vue.config.productionTip = false
new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})
```
#### 5、引入vuex
```
npm install vuex --save
```
> 在src下新建store/index.js
```
import Vue from "vue";
import Vuex from "vuex";
import {
    home
} from "./home";
import {
    first
} from "./first";

Vue.use(Vuex);

const store = new Vuex.Store({
    modules: {
        home,
        first
    }
});

export default store;
```
> home.js
```
import get from "lodash-es/get";

export const state = {
  username: "",
};

const getters = {
    username: state => get(state, ["username"]),
};

const mutations = {
  setUsername(state, payload) {
    state.username = payload;
  }
};

const actions = {};

export const home = {
  state,
  getters,
  mutations,
  actions,
  namespaced: true
};
```

> first.js
```
import get from "lodash-es/get";

export const state = {
  time: "",
};

const getters = {
    time: state => get(state, ["time"]),
};

const mutations = {
  setTime(state, payload) {
    state.time = payload;
  }
};

const actions = {};

export const first = {
  state,
  getters,
  mutations,
  actions,
  namespaced: true
};
```

> 修改main.js
```
import Vue from 'vue'
import App from './App.vue'
import router from './route' //路由设置
import store from './store' //路由设置

Vue.config.productionTip = false
new Vue({
  el: '#app',
  router,
  store,
  components: { App },
  template: '<App/>'
})
```
> 修改home/index.vue
```
<template>
  <div id="app">
    <h4>{{ msg }}</h4>
    <p>{{username}}</p>
    <section @click="change">change username</section>
    <img src="../../assets/image/back.png" alt>
  </div>
</template>

<script>
import { mapState,mapMutations } from "vuex";
export default {
  name: "home",
  data() {
    return {
      msg: "首页HOME欢迎您!"
    };
  },
  computed: {
    // 使用对象展开运算符将此对象混入到外部对象中
    ...mapState('home',['username'])
  },
  created () {
    console.log(this.username)
  },
  methods: {
    ...mapMutations('home',[
      'setUsername'
    ]),
    change(){
      this.setUsername('changeName')
    }
  }
};
</script>

<style lang="less" rel="stylesheet/less" scoped>
</style>
```
> 修改first/index.vue
```
<template>
  <div id="app">
    <h4>{{ msg }}</h4>
    <p>{{first}}</p>
    <section @click="change">change first</section>
    <img src="../../assets/image/red.png" alt>
  </div>
</template>

<script>
import { createNamespacedHelpers } from 'vuex'
const { mapState, mapMutations } = createNamespacedHelpers('first')

export default {
  name: "first",
  data() {
    return {
      msg: "first! page"
    };
  },
  computed: {
    // 使用对象展开运算符将此对象混入到外部对象中
    ...mapState(['first'])
  },
  created() {
    console.log(this.first);
  },
  methods: {
    ...mapMutations([
      'setFirst'
    ]),
    change() {
      this.setFirst('changeFirst')
    }
  }
};
</script>

<style lang="less" rel="stylesheet/less" scoped>
</style>
```
#### 6、使用lint
```
npm install -D eslint eslint-loader
npm install --save-dev babel-eslint
npm i -g eslint
eslint --init   //生产默认lint文件


CNyaya:webpack-4-tutorial yaya$ eslint --init
? How would you like to use ESLint? To check syntax, find problems, and enforce code style
? What type of modules does your project use? JavaScript modules (import/export)
? Which framework does your project use? Vue.js
? Where does your code run? (Press <space> to select, <a> to toggle all, <i> to invert selection)Browser
? How would you like to define a style for your project? Use a popular style guide
? Which style guide do you want to follow? Standard (https://github.com/standard/standard)
? What format do you want your config file to be in? JavaScript
Checking peerDependencies of eslint-config-standard@latest
The config that you've selected requires the following dependencies:

eslint-plugin-vue@latest eslint-config-standard@latest eslint@>=5.0.0 eslint-plugin-import@>=2.13.0 eslint-plugin-node@>=7.0.0 eslint-plugin-promise@>=4.0.0 eslint-plugin-standard@>=4.0.0
? Would you like to install them now with npm? Yes
Installing eslint-plugin-vue@latest, eslint-config-standard@latest, eslint@>=5.0.0, eslint-plugin-import@>=2.13.0, eslint-plugin-node@>=7.0.0, eslint-plugin-promise@>=4.0.0, eslint-plugin-standard@>=4.0.0
npm WARN webpack-4-tutorial@1.0.0 No description
npm WARN webpack-4-tutorial@1.0.0 No repository field.

+ eslint@5.16.0
+ eslint-plugin-standard@4.0.0
+ eslint-plugin-promise@4.1.1
+ eslint-config-standard@12.0.0
+ eslint-plugin-vue@5.2.2
+ eslint-plugin-node@8.0.1
+ eslint-plugin-import@2.16.0
added 33 packages and updated 1 package in 11.983s
An error occurred while generating your JavaScript config file. A config file was still generated, but the config file itself may not follow your linting rules.
Error: Cannot find module 'eslint-config-standard'
Referenced from: 
Error: An error occurred while generating your JavaScript config file. A config file was still generated, but the config file itself may not follow your linting rules.
Error: Cannot find module 'eslint-config-standard'
Referenced from: 
    at ModuleResolver.resolve (/Users/yaya/.nvm/versions/node/v8.11.3/lib/node_modules/eslint/lib/util/module-resolver.js:72:19)
    at resolve (/Users/yaya/.nvm/versions/node/v8.11.3/lib/node_modules/eslint/lib/config/config-file.js:507:28)
    at load (/Users/yaya/.nvm/versions/node/v8.11.3/lib/node_modules/eslint/lib/config/config-file.js:579:26)
    at configExtends.reduceRight (/Users/yaya/.nvm/versions/node/v8.11.3/lib/node_modules/eslint/lib/config/config-file.js:453:36)
    at Array.reduceRight (<anonymous>)
    at applyExtends (/Users/yaya/.nvm/versions/node/v8.11.3/lib/node_modules/eslint/lib/config/config-file.js:431:26)
    at Object.loadObject (/Users/yaya/.nvm/versions/node/v8.11.3/lib/node_modules/eslint/lib/config/config-file.js:566:35)
    at new Config (/Users/yaya/.nvm/versions/node/v8.11.3/lib/node_modules/eslint/lib/config.js:85:46)
    at new CLIEngine (/Users/yaya/.nvm/versions/node/v8.11.3/lib/node_modules/eslint/lib/cli-engine.js:466:23)
    at writeJSConfigFile (/Users/yaya/.nvm/versions/node/v8.11.3/lib/node_modules/eslint/lib/config/config-file.js:295:24)
CNyaya:webpack-4-tutorial yaya$ npm install --save-dev babel-eslint
npm WARN webpack-4-tutorial@1.0.0 No description
npm WARN webpack-4-tutorial@1.0.0 No repository field.

+ babel-eslint@10.0.1
added 2 packages in 9.886s
```

> .eslintrc.js
```
module.exports = {
    "env": {
        "browser": true,
        "es6": true,
        "node": true,
    },
    "extends": "standard",
    "globals": {
        "Atomics": "readonly",
        "SharedArrayBuffer": "readonly"
    },
    "parser": 'babel-eslint',
    "parserOptions": {
        "ecmaVersion": 6,
        "sourceType": "module",
        "allowImportExportEverywhere": true,
    },
    "plugins": [
        "vue","html"
    ],
    "rules": {
    }
};
```
> 配置vue的lint
```
npm i -D eslint-plugin-vue
npm install -D stylelint-webpack-plugin stylelint-config-standard
```
> .eslintrc.js    https://stylelint.io/user-guide/configuration/
```
module.exports = {
    "env": {
        "browser": true,
        "es6": true
    },
    "extends": [
        "plugin:vue/essential", "standard"
    ],
    "globals": {
        "Atomics": "readonly",
        "SharedArrayBuffer": "readonly"
    },
    "parserOptions": {
        "ecmaVersion": 2018,
        "sourceType": "module"
    },
    "plugins": [
        "vue"
    ],
    "rules": {
        "no-new": "off"
    }
};
```
> 新建 .stylelintrc
```
{
  "extends": "stylelint-config-standard",
  "rules": {
    "at-rule-empty-line-before": null
  }
}
```
> 修改package.json
```
"scripts": {
    "dev": "webpack-dev-server --config ./build/webpack.dev.js --colors",
    "nodemon": "nodemon --config nodemon",
    "build": "webpack  --config ./build/webpack.build.js",
    "lint": "eslint src --ext .js,.vue --cache --fix",
    "stylelint": "stylelint 'src/**/*.vue,src/style/**/*.less' --fix"
  },
  "pre-commit": [
    "lint",
    "stylelint"
  ],
```
> 修改webpack.base.js 
```
const StyleLintPlugin = require('stylelint-webpack-plugin');
module.exports = {
  // ... 其它选项
  plugins: [
    new StyleLintPlugin({
      files: ['**/*.{vue,htm,html,css,sss,less,scss,sass}'],
    })
  ]
}
```
> 让js和css输出到指定文件夹,修改webpack.prod.js
```
const merge = require('webpack-merge')
const MiniCssExtractPlugin = require("mini-css-extract-plugin"); //这个插件是为了取代extract-plugin, 并带来兼容性的改变
const baseWebpackConfig = require('./webpack.base')
//清空文件夹,By default, this plugin will remove all files inside webpack's output.path directory, as well as all unused webpack assets after every successful rebuild.
const utils = require('./utils.js')
const CleanWebpackPlugin = require('clean-webpack-plugin');
const devWebpackConfig = merge(baseWebpackConfig, {
    mode: "production",
    output:{
        publicPath: '/', //设置公共资源引入路径
        filename: utils.assetsPath('js/[name].[chunkhash].js'),
    },
    plugins:[
        new CleanWebpackPlugin(),
        // new ExtractTextPlugin(utils.assetsPath('css/[name].[contenthash].css')),
        new MiniCssExtractPlugin({
            filename: utils.assetsPath('css/[name].[contenthash].css'),
        }),
    ]
})

module.exports = devWebpackConfig
```
#### 7、编译公共组件
```
optimization: {
  splitChunks: {
    cacheGroups: {
      vendors: {
        name: 'chunk-vendors',
        test: /[\\\/]node_modules[\\\/]/,
        priority: -10,
        chunks: 'initial'
      },
      common: {
        name: 'chunk-common',
        minChunks: 1,
        priority: -20,
        chunks: 'initial',
        reuseExistingChunk: true
      }
    }
  }
}
```
> 参考 http://note.youdao.com/noteshare?id=0665924109c6a3aee48c8dd03902a70a

#### 8、手机端自适应
```
npm i -D postcss-px2rem fastclick
npm - lib-flexible
```
> 修改postcss.config.js
```
module.exports = {
  plugins: [
    require('autoprefixer')({
      browsers: ['last 10 Chrome versions', 'last 5 Firefox versions', 'Safari >= 6', 'ie> 8']
    }),
    require("postcss-px2rem")({
      baseDpr: 1, // base device pixel ratio (default: 2)
      threeVersion: false, // whether to generate @1x, @2x and @3x version (default: false)
      remVersion: true, // whether to generate rem version (default: true)
      remUnit: 37.5, // rem unit value (default: 75)
      remPrecision: 6, // rem precision (default: 6)
    })
  ]
}
```
> 修改src/main.js
```
import Vue from 'vue'
import "lib-flexible";//手机端自适应 px转rem
const FastClick = require("fastclick"); //解决300ms延迟

import App from './App.vue'
import router from './route' // 路由设置
import store from './store' // 路由设置


window["FastClick"] = FastClick;
Vue.config.productionTip = false


new Vue({
  el: '#app',
  router,
  store,
  components: { App },
  template: '<App/>'
})
```

> index.html增加头部
```
<html>

<head>
  <meta charset="utf-8">
  <!-- <meta name="viewport" content="width=device-width,initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no"> -->
  <meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">
  <link rel="stylesheet" href="<%=htmlWebpackPlugin.files.chunks.main.css %>">
  <meta name="format-detection" content="telephone=no, email=no">
  <meta name="screen-orientation" content="portrait">
</head>

<body>
  <div id="app">Hello, world!</div>
  <script>
    if ('addEventListener' in document) {
      document.addEventListener('DOMContentLoaded', function () {
        FastClick.attach(document.body)
      }, false)
    }
  </script>
  <script src="<%= htmlWebpackPlugin.files.chunks.main.entry %>"></script>
</body>

</html>
```
> TODO——验证scale！==1     淘宝固定1，美团固定0.5

#### 9、非编译文档，打包时复制到最终文件夹  
```
npm i -D  copy-webpack-plugin
mkdir static
```  
> 修改webpack.base.js
```
new CopyWebpackPlugin([
  {
    from: path.resolve(__dirname, '../static'),
    to: path.resolve(__dirname, '../dist/public')
  }
])
```

#### 10、commit时，只处理本次修改的文件
```
npm install -D husky lint-staged
```

> 修改 package.json
```
// "pre-commit": [
  //   "lint",
  //   "stylelint"
  // ],
  "scripts": {
    "dev": "webpack-dev-server --config ./build/webpack.dev.js --colors",
    "nodemon": "nodemon --config nodemon",
    "build": "webpack  --config ./build/webpack.build.js",
    "lint": "eslint src --ext .js,.vue --cache --fix",
    "stylelint": "stylelint 'src/**/*.vue,src/style/**/*.less' --fix"
  },
  "lint-staged": {
    "src/**/*.{js,vue}": [
      "npm run lint",
      "git add"
    ],
    "src/**/*.{less,vue}": [
      "npm run stylelint",
      "git add"
    ]
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "pre-push": "yarn lint && yarn test"
    }
  },
```

#### 11、组件库storybooks 

> https://storybook.js.org/docs/guides/guide-vue/ , 示例 https://storybooks-vue.netlify.com/?path=/story/custom-decorator-for-vue--render 
```
npm install @storybook/vue --save-dev
```
> Make sure that you have vue, vue-loader, vue-template-compiler, @babel/core, babel-loader and babel-preset-vue in your dependencies as well, because we list these as a peer dependencies:
```
npm install vue --save
npm install vue-loader vue-template-compiler @babel/core babel-loader babel-preset-vue --save-dev 
```
> 配置package.json
```
{
  "scripts": {
    "storybook": "start-storybook"
  }
}
```
- a、配置端口  指定配置文件，配置文件需要是文件夹内部的config.js
```
"storybook": "start-storybook -p 9001 -c .storybook",
```

- b、将组建打包成静态页面，可以单独访问
```
"build-storybook": "build-storybook -c .storybook -o .storybook.out"
```
- c、配置文件 .storybook/config.js
```
import {
  configure,
  addParameters,
  addDecorator
} from '@storybook/vue';
import {
  INITIAL_VIEWPORTS
} from '@storybook/addon-viewport';
// import '@storybook/addon-console';
import {
  setConsoleOptions
} from '@storybook/addon-console';

// addDecorator((storyFn, context) => withConsole()(storyFn)(context));
setConsoleOptions({
  options: {
    hierarchyRootSeparator: /\|/, //标题分类
  },
  panelExclude: [],
});

const newViewports = {
  kindleFire2: {
    name: 'Kindle Fire 2',
    styles: {
      width: '600px',
      height: '963px',
    },
  },
  kindleFireHD: {
    name: 'Kindle Fire HD',
    styles: {
      width: '533px',
      height: '801px',
    },
  },
};
addParameters({
  //配置背景色选项
  backgrounds: [{
      name: 'white',
      value: 'white',
      default: true
    },
    {
      name: 'facebook',
      value: '#3b5998'
    },
    {
      name: 'yellow',
      value: 'yellow'
    },
  ],
  //自定义视图大小
  viewport: {
    viewports: {
      ...INITIAL_VIEWPORTS,
      ...newViewports
    },
  }
});

function importAll(r) {
  r.keys().forEach(r)
}

//待处理的 storybook文件
function loadStories() {
  importAll(require.context("../stories", true, /\.story\.js$/));
  importAll(require.context("../src/components", true, /\.story\.js$/));
}

configure(loadStories, module);
```
- d、引入插件  .storybook/addon.js  ,插件引入顺序决定了 界面展示顺序
```
import '@storybook/addon-notes/register'; //添加备注
import '@storybook/addon-storysource/register';//显示源码
import '@storybook/addon-knobs/register';//显示props字段并可以更改
import '@storybook/addon-backgrounds/register';//设置背景色
import '@storybook/addon-viewport/register';//设置自定义屏幕
import '@storybook/addon-actions/register';
import '@storybook/addon-links/register';
```
- e、写组件
```
import {
  storiesOf
} from '@storybook/vue';

import {
  linkTo
} from '@storybook/addon-links';

import MyButton from './Button.vue';

import {
  withKnobs,
  text,
  number,
  boolean,
  array,
  select,
  color,
  date,
  button
} from '@storybook/addon-knobs';

import Vue from 'vue';
Vue.component('MyButton', MyButton);


storiesOf('Addon|Notes', module)
  .add('Default', () => ({
    template: '<MyButton>addon notes</MyButton>'
  }), {
    notes: 'You can write anything about this component'
  })

storiesOf('Addon|Links', module)
  .add('Go to Compents|Button', () => ({
    template: `<MyButton :rounded="true" @click-btn="click">addon Links</MyButton>`,
    methods: {
      click(val){
        console.log('click---'+val)
        linkTo('Compents|Button','custom button')
      } 
    },
  }))

storiesOf('Addon|Backgrounds', module)
  .add('Default', () => ({
    template: '<MyButton>addon backgrounds</MyButton>'
  }), {
    notes: 'with emoji-----backgrounds',
    backgrounds: [{
      name: 'grey',
      value: '#eeeeee',
      default: true
    }]
  })

storiesOf('Addon|Viewport', module)
  .add('Default', () => ({
    template: '<MyButton :rounded="true">addon viewport</MyButton>'
  }), {
    viewport: {
      defaultViewport: 'iphonex'
    }
  })


storiesOf('Addon|Knobs', module)
  .addDecorator(withKnobs)
  .add('All knobs', () => {
    const fruits = {
      Apple: 'apples',
      Banana: 'bananas',
      Cherry: 'cherries',
    };

    button('Arbitrary action', 'You clicked it!');

    return {
      props: {
        name: {
          default: text('Name', 'Jane')
        },
        stock: {
          default: number('Stock', 20, {
            range: true,
            min: 0,
            max: 30,
            step: 5,
          }),
        },
        fruit: {
          default: select('Fruit', fruits, 'apples')
        },
        price: {
          default: number('Price', 2.25)
        },
        colour: {
          default: color('Border', 'deeppink')
        },
        today: {
          default: date('Today', new Date('Jan 20 2017 GMT+0'))
        },
        // this is necessary, because we cant use arrays/objects directly in vue prop default values
        // a factory function is required, but we need to make sure the knob is only called once
        items: {
          default: (items => () => items)(array('Items', ['Laptop', 'Book', 'Whiskey']))
        },
        nice: {
          default: boolean('Nice', true)
        },
      },
      data: () => ({
        dateOptions: {
          year: 'numeric',
          month: 'long',
          day: 'numeric',
          timeZone: 'UTC'
        },
      }),
      computed: {
        stockMessage() {
          return this.stock ?
            `I have a stock of ${this.stock} ${this.fruit}, costing $${this.price} each.` :
            `I'm out of ${this.fruit}${this.nice ? ', Sorry!' : '.'}`;
        },
        salutation() {
          return this.nice ? 'Nice to meet you!' : 'Leave me alone!';
        },
        formattedDate() {
          return new Date(this.today).toLocaleDateString('en-US', this.dateOptions);
        },
        style() {
          return {
            'border-color': this.colour,
          };
        },
      },
      template: `
          <div style="border: 2px dotted; padding: 8px 22px; border-radius: 8px" :style="style">
            <h1>My name is {{ name }},</h1>
            <h3>today is {{ formattedDate }}</h3>
            <p>{{ stockMessage }}</p>
            <p>Also, I have:</p>
            <ul>
              <li v-for="item in items" :key="item">{{ item }}</li>
            </ul>
            <p>{{ salutation }}</p>
          </div>
        `,
    };
  });
```



- f、自定义打包编译
> Create a .storybook/webpack.config.js file.  
> 查看默认的webpack配置,Edit it’s contents:
```
module.exports = async ({ config }) => console.dir(config.plugins, { depth: null }) || config;
```
> Then run storybook:
```
yarn storybook --debug-webpack
```
> 增加less
```
const path = require('path');

// Export a function. Accept the base config as the only param.
module.exports =  async ({
  config,
  mode
}) => {
  // `mode` has a value of 'DEVELOPMENT' or 'PRODUCTION'
  // You can change the configuration based on that.
  // 'PRODUCTION' is used when building the static version of storybook.

  // Make whatever fine-grained changes you need
  config.module.rules.push({
    test: /\.less$/,
    use: ['style-loader', 'css-loader', 'postcss-loader','less-loader'],
    include: path.resolve(__dirname, '../'),
  });
  config.module.rules.push({
    test: /\.stories\.jsx?$/,
    loaders: [
      {
        loader: require.resolve('@storybook/addon-storysource/loader'),
        options: { injectDecorator: false },
      },
    ],
    enforce: 'pre',
  });

  // Return the altered config
  return config;
};
```

- g、一键安装
```
mkdir storybook-vue
cd storybook-vue/
npm init -y
npx -p @storybook/cli sb init --type vue
npm run storybook


npm i -D vue-loader @storybook/addon-storysource
npm i vue
```


#### 12、多页面
> 在这个过程中，需要配置多个入口entry，所以文件src下面的目录以及打包配置文件都发生了一些变动； 

> 为了实现每个页面的title的独立性，最初考虑使用.ejs模板，但是一直报编译错误，最后发现使用.html也是可以和webpack-html-plugin共同实现这个功能，最终使用了.html作为编译模板

###### 同时，遇到了css编译突然报错，需要删除.html文件的被注释的代码

> 在根目录创建appConfig.js
```
// 页面配置
exports.pages = [{
    filename: "single",
    title: "关于" // title 可以在模板中指定,
  },
  {
    filename: "default",
    title: "主页" // title 可以在模板中指定,
  }
];
```
> utils.js 增加entry、htmlPlugins入口
```
//多页面入口
exports.entry = (function () {
  var entry = {};
  glob.sync("./src/pages/*").forEach(name => {
    var n = name.slice(12, name.length - 0);
    appConfig.pages.forEach(function (page) {
      if (page.filename === n) {
        entry[n] = name + "/entry.js";
      }
    });
  });
  return entry;
})();

exports.htmlPlugins = function () {
  let htmlPlugins = []
  appConfig.pages.forEach(function (page) {
    var conf = {
      template: page.template || "./index.html",
      title: page.title || "currentPage",
      filename: page.filename + ".html",// 生成的html存放路径,文件名，相对于path 
      chunks: ["chunk-vendors", page.filename],
      inject: true, //js插入的位置
      // hash: true,  
      minify: { // 压缩HTML文件
        removeComments: true, // 移除HTML中的注释
        collapseWhitespace: true, // 删除空白符与换行符
        removeAttributeQuotes: false
      },
      // RUN_ENV: utils.env
    }; // 移除HTML中的注释 // 删除空白符与换行符
    htmlPlugins.push(new HtmlWebpackPlugin(conf));

  return htmlPlugins
}
```
> 修改index.html,增加自定义title
```
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">
  <meta name="format-detection" content="telephone=no, email=no">
  <meta name="screen-orientation" content="portrait">
  <link rel="shortcut icon" href="/static/favicon.ico" type=image/x-icon />
  <title><%= htmlWebpackPlugin.options.title %></title>
</head>

<body>
  <div id="app">Hello, world!</div>
  <script>
    if ('addEventListener' in document) {
      document.addEventListener('DOMContentLoaded', function () {
        FastClick.attach(document.body)
      }, false)
    }
  </script>
</body>

</html>
```
> 修改webapck.base.js,设置公用的entry和html打包
```
const path = require('path');
const WebpackMd5Hash = require('webpack-md5-hash'); //更改文件，更改hash名称
const VueLoaderPlugin = require('vue-loader/lib/plugin'); //编译vue
const StyleLintPlugin = require('stylelint-webpack-plugin'); //校验vue的style格式
const CopyWebpackPlugin = require('copy-webpack-plugin');
const utils = require('./utils.js')

module.exports = {
    entry: utils.entry,
    output: {
        path: path.resolve(__dirname, '../dist'),
    },
    module: {
        rules: [
            {
                test: /\.vue$/,
                loader: 'vue-loader'
            },
            {
                test: /.js$/,
                exclude: path.resolve(__dirname, '../node_modules'),
                use: {
                    loader: "babel-loader"
                }
            },
            {
                test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
                use: {
                    loader: 'url-loader',
                    query: {
                        limit: 10000,
                        name: utils.assetsPath('img/[name].[hash:7].[ext]')
                    }
                }
            },
            {
                test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
                use: {
                    loader: 'url-loader',
                    query: {
                        limit: 10000,
                        name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
                    }
                }
            },
        ]
    },
    resolve: {
        extensions: [".js", ".json", ".jsx", ".css", ".less", ".vue"],
        // 使用的扩展名
        alias: {
            'vue$': 'vue/dist/vue.esm.js', // 用 webpack 1 时需用 'vue/dist/vue.common.js'
            '@': path.resolve(__dirname, '../src'),
        },
    },
    plugins: [
        new VueLoaderPlugin(),
        new StyleLintPlugin({
            files: [path.resolve(__dirname, '../src/**/*.{vue,htm,html,css,sss,less,scss,sass}'),'../src/**/*.{vue,htm,html,css,sss,less,scss,sass}'],
        }),
        new WebpackMd5Hash(),
        new CopyWebpackPlugin([
          {
            from: path.resolve(__dirname, '../static'),
            to: path.resolve(__dirname, '../dist/static')
          }
        ]),
        ...utils.htmlPlugins(),
    ]
};
```
> webapack.dev.js
```
const merge = require('webpack-merge')
const webpack = require('webpack');
const baseWebpackConfig = require('./webpack.base')
const path = require('path');

const devWebpackBase = {
  mode: "development",
  output: {
      publicPath: '/', //设置公共资源引入路径
      filename: '[name].js',
  },
  devtool: '#eval-source-map',
  devServer: {
      // contentBase: path.join(__dirname, './dist'),
      compress: true, //启用压缩
      hot: true, //启用 webpack 的 模块热替换 功能
      hotOnly: true,
      port: 9000,
      open: true, //告诉 dev-server 在 server 启动后打开浏览器。默认禁用
  },
  module: {
      rules: [{
          enforce: 'pre',
          test: /\.(js|vue)$/,
          loader: 'eslint-loader',
          include: [path.resolve(__dirname, '../src')], // 指定检查的目录
          exclude: /node_modules/,
          options: {
            fix: true
          }
      },
      {
        test: /\.css$/,
        use: [
          'vue-style-loader',
          'css-loader'
        ]
      },
      {
        test: /\.less$/,
        use: [
          'vue-style-loader',
          'css-loader',
          'postcss-loader', 
          'less-loader'
        ]
      }]
  },
  plugins: [
      new webpack.HotModuleReplacementPlugin(),
  ]
}

const devWebpackConfig = merge(baseWebpackConfig,devWebpackBase )

module.exports = devWebpackConfig
```
> webapck.prod.js 
```
const merge = require('webpack-merge')
const MiniCssExtractPlugin = require("mini-css-extract-plugin"); //这个插件是为了取代extract-plugin, 并带来兼容性的改变
const baseWebpackConfig = require('./webpack.base')
//清空文件夹,By default, this plugin will remove all files inside webpack's output.path directory, as well as all unused webpack assets after every successful rebuild.
const utils = require('./utils.js')
const CleanWebpackPlugin = require('clean-webpack-plugin');
const devWebpackConfig = merge(baseWebpackConfig, {
    mode: "production",
    output:{
        publicPath: '/', //设置公共资源引入路径
        filename: utils.assetsPath('js/[name].[chunkhash].js'),
    },
    optimization: {
      minimize: false,
      splitChunks: {
        cacheGroups: {
          vendors: {
            name: 'chunk-vendors',
            test: /[\\/]node_modules[\\/]/,
            priority: -10,
            chunks: 'all'
          },
          common: {
            name: 'chunk-common',
            minChunks: 2,
            priority: -20,
            chunks: 'all',
            reuseExistingChunk: true
          }
        }
      }
    },
    module: {
      rules:[
        {
          test: /\.css$/,
          use: [
            MiniCssExtractPlugin.loader,
            'css-loader'
          ]
        },
        {
          test: /\.less$/,
          use: [
            MiniCssExtractPlugin.loader,
            'css-loader',
            'postcss-loader', 
            'less-loader'
          ]
        }
      ]
    },
    plugins:[
        new CleanWebpackPlugin(),
        new MiniCssExtractPlugin({
            filename: utils.assetsPath('css/[name].[contenthash].css'),
        }),
    ]
})

module.exports = devWebpackConfig
```
> 目录结果发生了一些变化，参考  

![目录](https://note.youdao.com/yws/public/resource/e50c91c7b2dbed81cf6346913c1e1810/xmlnote/91E16DE203694AEE84B1BEB2BC42C417/3084)


#### 13、测试
> 安装,参考 [vue-test配置]:https://vue-test-utils.vuejs.org/zh/guides/testing-single-file-components-with-jest.html
```
npm install --save-dev jest @vue/test-utils
npm install --save-dev vue-jest
npm install --save-dev babel-jest
npm install --save-dev jest-serializer-vue
```
> 在根目录新建test文件夹和jest.config.js
```
module.exports = {
  "moduleFileExtensions": [
    "js",
    "json",
    // 告诉 Jest 处理 `*.vue` 文件
    "vue"
  ],
  "transform": {
    // 用 `vue-jest` 处理 `*.vue` 文件
    ".*\\.(vue)$": "vue-jest",
    // 用 `babel-jest` 处理 js
    "^.+\\.js$": "<rootDir>/node_modules/babel-jest"
  },
  // 支持源代码中相同的 `@` -> `src` 别名
  "moduleNameMapper": {
    "^@/(.*)$": "<rootDir>/src/$1"
  },
  "collectCoverage": true,
  "collectCoverageFrom": ["src/**/*.{js,vue}", "!**/node_modules/**"],
  // "coverageReporters": ["html", "text-summary"],//默认格式的测试覆盖率报告
  // 快照的序列化工具
  // "snapshotSerializers": ["jest-serializer-vue"],
}
```
> package.json
```
{
  "scripts": {
    ...,
    "test": "jest"
  }
}
```
> 搭建测试目录 

![测试目录](https://note.youdao.com/yws/public/resource/e50c91c7b2dbed81cf6346913c1e1810/xmlnote/35E17E6BB2F24AE39A1935AAA28A0B97/3119)

> test/unit/pages/homepage/App.spec.js 
```import { createLocalVue, mount } from '@vue/test-utils'
import APP from '../../../../src/pages/homepage/App.vue'

import VueRouter from 'vue-router'

const localVue = createLocalVue()
localVue.use(VueRouter)

const routes = [{ path: '/foo' }]

const router = new VueRouter({
  routes
})

describe('Component', () => {
  test('is a Vue instance', () => {
    const wrapper = mount(APP,{
      localVue,
      router
    })
    expect(wrapper.isVueInstance()).toBeTruthy()
  })
})

```
> npm run test 
```
CNyaya:webpack-4-tutorial yaya$ npm run test

> webpack-4-tutorial@1.0.0 test /Users/yaya/Documents/learn/case/webpack-4-tutorial
> jest

 PASS  test/unit/pages/homepage/App.spec.js
  Component
    ✓ is a Vue instance (20ms)

------------------------|----------|----------|----------|----------|-------------------|
File                    |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
------------------------|----------|----------|----------|----------|-------------------|
All files               |        1 |        0 |     2.56 |        1 |                   |
 components/Button      |        0 |        0 |        0 |        0 |                   |
  index.story.js        |        0 |      100 |        0 |        0 |... 21,26,46,59,73 |
  index.vue             |        0 |        0 |        0 |        0 |    42,55,65,69,74 |
 pages/homepage         |     12.5 |      100 |       50 |     12.5 |                   |
  App.vue               |       50 |      100 |       50 |       50 |                31 |
  entry.js              |        0 |      100 |      100 |        0 |    10,12,13,14,16 |
  index.js              |        0 |      100 |      100 |        0 |                 2 |
 pages/homepage/cropper |        0 |      100 |        0 |        0 |                   |
  index.vue             |        0 |      100 |        0 |        0 |    28,35,51,55,58 |
 pages/homepage/date    |        0 |      100 |        0 |        0 |                   |
  index.vue             |        0 |      100 |        0 |        0 |... ,98,99,100,103 |
 pages/homepage/first   |        0 |      100 |        0 |        0 |                   |
  index.vue             |        0 |      100 |        0 |        0 |... 28,29,30,31,36 |
 pages/homepage/home    |        0 |      100 |        0 |        0 |                   |
  index.vue             |        0 |      100 |        0 |        0 |... 25,26,27,28,35 |
 pages/homepage/route   |        0 |      100 |      100 |        0 |                   |
  index.js              |        0 |      100 |      100 |        0 |        6,7,8,9,11 |
 pages/homepage/store   |        0 |      100 |        0 |        0 |                   |
  first.js              |        0 |      100 |        0 |        0 | 3,7,8,11,13,17,19 |
  home.js               |        0 |      100 |        0 |        0 | 3,7,8,11,13,17,19 |
  index.js              |        0 |      100 |      100 |        0 |             10,12 |
 pages/single           |        0 |      100 |        0 |        0 |                   |
  app.vue               |        0 |      100 |        0 |        0 |                11 |
  entry.js              |        0 |      100 |      100 |        0 |              6,13 |
  router.js             |        0 |      100 |        0 |        0 |      4,9,10,12,29 |
 pages/single/views     |        0 |      100 |        0 |        0 |                   |
  about.vue             |        0 |      100 |        0 |        0 |             54,59 |
  protocol.vue          |        0 |      100 |        0 |        0 |           222,227 |
------------------------|----------|----------|----------|----------|-------------------|
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        3.055s
Ran all test suites.

```

#### 14、typeScript
> 参考 [TypeScript-Vue-Starter]:https://github.com/Microsoft/TypeScript-Vue-Starter#typescript-vue-starter  

```
npm install --save-dev typescript webpack ts-loader css-loader vue vue-loader vue-template-compiler
yarn add --dev vue-class-component
yarn add --dev tslint-loader  tslint
yarn add --dev babel-plugin-syntax-jsx babel-plugin-transform-vue-jsx babel-helper-vue-jsx-merge-props 
npm install --save vuex-class
npm install typescript typescript-eslint-parser --save-dev
npm install eslint-plugin-typescript --save-dev
npm install --save-dev eslint typescript typescript-eslint-parser eslint-plugin-typescript eslint-config-alloy
```
> 修改webpack.base.js
```
{
  test: /\.tsx?$/,
    use: [{
        loader: "ts-loader",
        options: {
          transpileOnly: true,
          appendTsSuffixTo: [/\.vue$/]
        }
      },
      {
        loader: "tslint-loader",
        options: {
          enforce: "pre"
        }
      }
    ],
    include: [path.resolve(__dirname, '../src')],
    exclude: /node_modules/
},
```
> 在根目录新建tsconfig.json
```
{
  "compilerOptions": {
    "outDir": "./built/",
    "sourceMap": true,
    "noImplicitReturns": true,
    // 与 Vue 的浏览器支持保持一致
    "target": "es5",
    // 这可以对 `this` 上的数据属性进行更严格的推断
    "strict": true,
    // 如果使用 webpack 2+ 或 rollup，可以利用 tree-shake:
    "module": "es2015",
    "moduleResolution": "node"
  },
  "include": [
      "./src/**/*"
  ]
}
```
```
{
  "compilerOptions": {
    // "outDir": "./build/",//重定向输出目录,排除的目录
    "baseUrl": "./src",//解析非相对模块名的基准目录。查看 模块解析文档了解详情。
    "paths": {
      "@/*": ["*"]
    },
    "sourceMap": true,
    "noImplicitAny": false,//对包含任何类型的表达式和声明引发错误。
    "noImplicitReturns": true,//Report an error when not all code paths in function return a value.
    "allowJs": true,//允许编译javascript文件。
    "allowSyntheticDefaultImports": true,//允许从没有设置默认导出的模块中默认导入。这并不影响代码的输出，仅为了类型检查。
    "experimentalDecorators": true,//	启用实验性的ES装饰器。
    "emitDecoratorMetadata": true,//给源码里的装饰器声明加上设计类型元数据。
    "lib": ["dom",
      "es5",
      "es6",
      "es7",
      "es2015.promise"],
    // 与 Vue 的浏览器支持保持一致
    "target": "es5",
    // 这可以对 `this` 上的数据属性进行更严格的推断
    "strict": true,
    // 如果使用 webpack 2+ 或 rollup，可以利用 tree-shake:
    "module": "es2015",
    "moduleResolution": "node",
    "jsx": "preserve",
  },
  "include": [
    "./src"
  ],
  "exclude": [
    "node_modules",
    "**/*.spec.ts"
  ]
}
```

> 修改 src/pages/homepage/App.vue
```
<template>
  <div id="app">
    <h2>{{ msg }}</h2>
    <ul>
      <li>
        <router-link to="/">首页</router-link>
      </li>
      <li>
        <router-link to="/first">第一页</router-link>
      </li>
      <li>
        <router-link to="/date">日期插件</router-link>
      </li>
      <li>
        <router-link to="/cropper">图片裁剪</router-link>
      </li>
    </ul>
    <router-view/>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import Component from 'vue-class-component'

@Component
export default class App extends Vue {
  // initial data
  msg:string = 'Hello world!'

  // method
  a () {
    console.log('hahahah')
  }
}
</script>

<style lang="less" rel="stylesheet/less">
@import "./style/index.less";

#app {
  color: rgb(81, 218, 127);
}
</style>

```


> 修改 src/pages/homepage/home/index.vue
```
<script lang="ts">
import Vue from 'vue'
import Component from 'vue-class-component'
import {
  namespace
} from 'vuex-class'

import _ from 'lodash'
const home = namespace("home");


@Component({
  name: "home",
})
export default class Home extends Vue {
  @home.Mutation setUsername;
  @home.State username;
  // initial data
  msg = '首页HOME欢迎您!!!2233';

  // lifecycle hook
  created () {
    console.log(this.username)
    console.log(_.camelCase('Foo Bar'))
    console.log(_.camelCase('--foo-bar--'))
    console.log(_.camelCase('__FOO_BAR__'))
  }

  // method
  change () {
    this.setUsername('changeName')
  }
}

</script>
```
> 其余.vue修改类似

> 修改 src/pages/homepage/store/index.ts
```
import Vue from 'vue'
import Vuex from 'vuex'
import {
  home
} from './home'
import {
  first
} from './first'

Vue.use(Vuex)

const store = new Vuex.Store<RootState>({
  modules: {
    home,
    first,
  },
})

export default store

```

> 新增 src/pages/homepage/typed/store.d.ts
```
// Store
declare interface RootState {
  home: HomeState;
  first: FirstState;
}

declare interface HomeState {
  username: string;
}

declare interface FirstState {
  first: string;
}
```
> 修改 src/pages/homepage/store/first.ts
```
import { GetterTree, MutationTree, ActionTree, Module } from "vuex";
import get from 'lodash-es/get'

export const state:FirstState = {
  first: 'hello-username',
}

const getters:GetterTree<FirstState, RootState> ={
  first: state => get(state, ['first']),
}

const mutations: MutationTree<FirstState> ={
  setFirst(state:any, payload) {
    state.first = payload
  },
}

const actions : ActionTree<FirstState, RootState> = {}

export const first: Module<FirstState, RootState> = {
  state,
  getters,
  mutations,
  actions,
  namespaced: true,
}


```
> 修改 src/pages/homepage/store/home.ts
```
import { GetterTree, MutationTree, ActionTree, Module } from "vuex";
import get from 'lodash-es/get'

export const state:HomeState = {
  username: 'hello-username',
}

const getters:GetterTree<HomeState, RootState> ={
  username: state => get(state, ['username']),
}

const mutations: MutationTree<HomeState> ={
  setUsername(state:any, payload) {
    state.username = payload
  },
}

const actions : ActionTree<HomeState, RootState> = {}

export const home: Module<HomeState, RootState> = {
  state,
  getters,
  mutations,
  actions,
  namespaced: true,
}

```

##### bug处理  
> 1、Support for the experimental syntax 'decorators-legacy' isn't currently enabled  
```
npm install @babel/plugin-proposal-decorators --dev
```
> .babelrc
```
"plugins": [
  [
    "@babel/plugin-proposal-decorators",
    {
      "legacy": true
    }
  ]
],
```
> 2、Parsing error: Unexpected character '@'    (lint报错)
```
```
> 3、can't find App.vue,在src下面新建文件vue-shim.d.ts
```
declare module "*.vue" {
  import Vue from "vue"
  export default Vue
}
```
> 4、与eslint冲突
```
ERROR in ./src/pages/homepage/App.vue
Module Error (from ./node_modules/eslint-loader/index.js):

/Users/yaya/Documents/learn/case/webpack-4-tutorial/src/pages/homepage/App.vue
  26:0  error  Parsing error: Unexpected character '@'

✖ 1 problem (1 error, 0 warnings)
```
> 暂时的解决方法，取消了eslint,webpack.dev.js
```
module: {
      rules: [
      // {
      //     enforce: 'pre',
      //     test: /\.(js|vue)$/,
      //     loader: 'eslint-loader',
      //     include: [path.resolve(__dirname, '../src')], // 指定检查的目录
      //     exclude: /node_modules/,
      //     options: {
      //       fix: true
      //     }
      // },
      {
        test: /\.css$/,
        use: [
          'vue-style-loader',
          'css-loader'
        ]
      },
      {
        test: /\.less$/,
        use: [
          'vue-style-loader',
          'css-loader',
          'postcss-loader', 
          'less-loader'
        ]
      }]
  },
```

#### 15、服务端渲染



#### 16、axios封装    




#### 17、多语言







