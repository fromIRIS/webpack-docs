># getting started

## 欢迎

这个小教程带你过一遍简单的例子

你会学到：

 - 如何安装webpack
 - 如何使用webpack
 - 如何使用loaders
 - 如何使用开发服务器

## 安装webpack

你需要先安装nodejs

```
npm install webpack -g

```

> 这使得webpack全局命令可用

## 设置应用

从全新的目录开始。

> webpack会分析入口文件中的依赖文件。这些文件（也称模块）也会包含在`bundle.js`中。webpack会给每个模块一个独立的id然后把所有凭id访问的模块存到bundle.js中。只有入口模块在初始化时会被执行。一个小型的运行时会提供一个require函数，当需要时会执行依赖的资源。

## 第一个loader

我们想要给应用添加css文件。

webpack默认只支持javascript，所以我们需要css-loader来处理css文件。

执行`npm install css-loader style-loader`下载安装loaders（他们需要安装在本地，不带-g）这会创建一个loaders存放的`node_modules`文件夹。

开始使用他们：

```
require("!style!css!./style.css");
document.write(require("./content.js"))
```

> 在引入模块时添加loader的前缀，模块会经历一个loader管道，这些loaders将模块内容以特定的方式进行改变。在所有改变完成后，最后的结果是一个javascript模块。

## 绑定loaders

我们不想写如此长的引用`require("!style!css!./style.css")`

我们可以给loaders绑定拓展所以我们只用写`require("./style.css")`

```
require("./style.css");
document.write(require("./content.js"))
```

跑命令

```
webpack ./entry.js bundle.js --module-bind 'css=style!css'
```

## 配置文件

我们想把配置信息移到一个配置文件中：

添加配置文件webpack.config.js：

```
module.exports = {
    entry: "./entry.js",
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.css$/, loader: "style!css" }
        ]
    }
};
```

## 漂亮的输出

如果项目扩大了应用编译会变慢，所以我们想增加以下进度条，并且带有颜色。。

```
webpack --progress --colors
```

## 监控模式

我们不想每次改动代码都手动进行编译。。

```
webpack --progress --colors --watch
```

webpack能缓存未变化的模块，在每次编译后都能产出文件。

> 当使用监控模式，webpack对所有文件在编译过程都安装了文件监听器。如果任何改动被捕捉到了，webpack会再次编译。当缓存开启了，webpack会把每个模块都存在内存里，如果没有变动就会重复利用。

## 开发服务器

开发服务器是更棒的选择。

```
npm install webpack-dev-server -g
```

```
webpack-dev-server --progress --colors
```

这绑定了一个小型express服务器在localhost:8080。作为静态资源和bundle的服务器。当重新编译时浏览器会重新更新。打开 http://localhost:8080/webpack-dev-server/bundle 。

  dev 服务器会使用webpack的监控模型，这能防止webpack释放结果文件到硬盘上，安装它保持结果文件都是从内存中读取的。
