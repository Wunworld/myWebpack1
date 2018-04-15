# myWebpack1

工作流程记录：
### 1.初始化项目：npm init -y
### 2.安装webpack,vue,vue-loader
```javascript
npm install webpack vue vue-loader
```
### 3.按装之后根据警告提示，安装css-loader,vue-template-conpiler依赖包。
```javascript
npm install css-loader vue-template-compiler 
```
项目初始化基本完成。

一直遇到提示 install webpack-cli -D,即使安装了也没有用，索性直接删除了，之后可以使用。
```javascript
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
    "build": "webpack --config webpack.config.js"
    只有在这里面写webpack他才会调用这里面的webpack,否则会调用全局的webpack,会导致很多版本不同出错
  },

```
### 4.新建webpack.config.js
```javascript
const path = require("path");//nodejs中的基本包，处理路径的
module.exports = {
    entry: path.join(__dirname,"src/main.js"),//__dirname代表文件所在的目录
    output: {
        filename: "bundle.js",
        path: path.join(__dirname,"dist")
    }
}

```

### 5.新建src文件，源代码
src下新建app.vue
```javascrip
<template>
    <div id="text">{{text}}</div>
</template>

<script>
export default {
    data() {//数据
        return {
            text: "abc"
        };
    }
}
</script>


<style>

#text{color: red;}
</style>
```
src下新建main.js
```javascript
import Vue from "vue";
import App from "./app.vue";//.vue文件
//分别导进来文件

//创建根文件
const root = document.creatElement("div");
document.body.appendChild(root);
new Vue({
    render: (h) => h(App)//通过它挂载到html页面中
}).$mount(root);//挂载到html页面中
```
### 6.按需要添加loader
```javascript
module.exports = {
    entry: path.join(__dirname,"src/main.js"),//__dirname代表文件所在的目录
    output: {
        filename: "bundle.js",
        path: path.join(__dirname,"dist")
    },
    module: {
        rules: [
            {
                test: /\.vue$/,
                loader: "vue-loader"
            },
            {
                test:/\.styl/,//stylus预处理
                use: [
                    "style-loader",
                    "css-loader",
                    "stylus-loader"//专门处理stylus文件，处理完成之后让css-loader处理css,扔给上一层处理，自己处理自己。比较方面，可以不用写任意的括弧，标点符号。兼容css==== npm install stylus-loader stylus
                ]
            },
            {
                test: /\.css$/,
                use: [
                    "style-loader",
                    "css-loader"
                ]
            },
            {
                test: /\.(gif|jpg|jpeg|png|svg)$/,
                use: [
                    {
                        loader: "url-loader",//的安装依赖file-loader
                        options: {
                            limit: 1024,//如果文件小于1024就会把图片转译成base64的代码
                            name: "[name]-aa.[ext]"//指定输出的名字，[name].[ext],原来的名字.扩展名,-aa是自定义的=====之后把相应的loader安装即可。
                        }
                    }
                ]
            }
        ]
    }
}

```

webpack做的事情就是把不同的静态资源类型打包成一个js,在html中引用js，在html引用js即可。把零碎的js打包在一起减少http请求。使用模块依赖，这样积累，以后的项目可以复用。
### 7.在main.js中导入所需要的js,css,图片等模块。
```javascript
import Vue from "vue";
import App from "./app.vue";//.vue文件
import "./assert/style/style.css";
import "./assert/img/123.jpg";
...
```
### 8.配置webpack-dev-server
安装：
```javascript
npm install webpack-dev-server
```
安装之后在package.json中设置
```javascript
"scripts": {
    "dev": "webpack-dev-server --config webpack.config.js"
}
```
在webpack.config.js中全局的配置。
```javascript
module.exports = {
    target: "web",//编译目标是web平台
    entry: "..."
}
//判断那个环境
//安装一个cross-env,不同的平台上运行的环境变量不一样。
```
```javascript
"build": "cross-env NODE_ENV=production webpack --config webpack.config.js",//windows: set NODE_ENV=production
"dev": "cross-env NODE_ENV=development webpack-dev-server --config webpack.config.js"
"dev": "cross-env NODE_ENV=development webpack-dev-server --mode development --config webpack.config.js"//解决热跟新是保存页面多次插入模板
```
在webpack.config.js中判断：
```javascript
const isDev = process.env.NODE_ENV === "development";//判断是不是develment,在里面设置的环境变量都在process.env这个对象里
const config = {
    target: "web",//编译目标是web平台
    entry: "..."
}
if(isDev) {
    config.devServer = {//给config添加一个对象。server是webpack2.0
        port: "8000",
        host: "0.0.0.0",//可以通过localhost,127.0.0.1,访问，也可以手机测试，其他本机内网也可以访问。
        overlay: {
            errors: true,//编译时出现错误显示
        },
        //热加载hot,需要webpack的HotModuleReplacementPlugin插件
        hot: true//修改了一个组件的代码，至渲染组件的数据，不会整个页面刷新,安装插件
    },
    config.plugins.push(
        new webpack.HotModuleReplacementPlugin(),//启动这个插件，热加载
        new webpack.NoEmitOnErrorsPlugin()
    ),
    config.devtool = "#cheap-module-eval-source-map"//映射编译后的代码,(.vue,es6代码映射成浏览器识别的代码)
}
module.exports = config;//整体的暴露出去
```
### 9.安装html插件
html-webpack-plugin
webpack.config.js中全局引用：
```javascript
const HTMLPlugin = require("html-webpack-plugin");
const webpack = require("webpack");
//在config对象中添加
plugins: [
    new webpack.DefinePlugin({//自身自带的一个插件
        "process.env": {//判断是哪一个环境，开发环境还是生产环境
            NODE_ENV:isDev ? " 'development' " : " 'production' "//判断环境
        }
    }),
    new HTMLPlugin();//new 就可以了
]
```
### 安装postcss-loader autoprefixer babel-loader babel-core
安装之后新建.babelrc,postcss.config.js
postcss.config.js:
```javscript
const autoprefixer = require("autoprefixer");
module.exports = {
    plugins: [
        autoprefixer()
    ]
}
//后处理css,通过次组建处理css,处理浏览器前缀的等。
```
babelrc:
安装babel-preset-env babel-plugin-transform-vue-jsx
```javascript
{
    "presets": [
        "env"
    ],
    "plugins": [
        "transform-vue-jsx"//专门转化vue中的js
    ]
}
//如何使用jxs代码的识别，
```
在插件中添加rules
```javascript
{
    test: /\.jsx$/,
    loader: "babel-loader"
}
//需要在css-loader中添加一个对象
{
    "css-loader",
    {
        loader: "postcss-loader",
        options: {
            sourceMap: true
        }
    },//stylus-loader会自动生成sourse map,postcss-loader也会自动生成，这样前面的生成后后面的在不生成，使用效率会很好。使用效率会很好。
    "stylus-loader"
}
```
### .jsx文件
```javascript
import "../.css";

export default {
    data() {
        return {
            author: "intelwisd"
        }
    },
    render() {
    //可以使用一些js业务操作
        return (//返回标签
            <div>{this.author}</div>
        );
    }
}
```
使用导入组件相同。

### webpack正式环境打包的优化
把css单独的打包出来，安装extract-text-webpack-plugin
webpack.config.js:导入使用
```javascript
const ExtractPlugin = require("extract-text-webpack-plugin");//非js打包成一个静态文件，做浏览器缓存

//根据不同的环境去添加css 
if(isDev) {//开发时使用styl
    config.module.rules.push({
        test: /\.styl/,
        use:[
            "style-loader",
            "css-loader",
            {
                loader: "postcss-loader",
                options: {
                    sourceMap: true 
                }
            },
            'stylus-loader'
        ]
    });
}else{//线上时使用把css单独打包成静态文件
    config.module.rules.push({
        test: /\.styl/,
        use: ExtractPlugin.extract({
            fallback: "style-loader",
            use: [
                "css-loader",
                {
                    loader: "postcss-loader",
                    options: {
                        sourceMap: true
                    }
                },
                "stylus-loader"
            ]
        })
    }),
    config.plugins.push(
        new ExtractPlugin("styles.[contentHash:8].css")
    ),
    config.output.filename = "[name].[chunkhash:8].js"
    //接着处理filename
}
```
```javascript
filename: "bundle.[hash:8].js"//默认的
```
vue-loader会根据每个组件里面的样式显示的时候才会显示到页面上，好处是使用组件异步加载的时候css跟着异步加载，会省很多流量。

### 单独打包库代码以至于缓存
```javascript
else{//线上环境
    config.entry = {//entry路径下添加打包的库文件
        app: path.join(__dirname,"src/index.js"),
        vendor: ["vue","vue-router"]
    },
    config.plugin.push({//利用此插件
        new webpack.optimize.CommonsChunkPlugin({
            name: "vender"//库文件打包成的名字
        }),//这两者是有顺序的
        new webpack.optimize.CommonsChunkPlugin({
            name: "runtime"//webpack相关的文件打包到包度的文件中，好处：有新的模块时，webpack会添加一个新的id上去，就会导致打包时hash发生变化，会阻止浏览器的长缓存。
        })
    })
}
```
### hash和chunkhash
chunkhash: 在entry中生成的不同的节点,不然打都打包没有意义。线上环境必须用chunkhash。
hash: 所有打包出来的模块都是同一个哈希，整个应用的hash
