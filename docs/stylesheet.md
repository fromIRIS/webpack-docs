> # stylesheets

## 内联的样式

通过使用`style-loader`和`css-loader`将样式文件内嵌到webpack js打包文件中。通过这种方式你可以将你的样式文件和其他模块一样处理。引入样式如下方式`require("./stylesheet.css")`

### 安装

从npm中安装loaders

```
npm install style-loader css-loader --save-dev
```

### 配置

下面介绍一个使`require()`正常工作的配置例子

```
{
  module: {
    loaders: [
      {test: /\.css$/, loader: "style-loader!css-loader"}
    ]
  }
}
```

> 杜宇预编译的css语言可以查询对应的loader的配置例子，你可以在module中加入

请牢记管理modules的执行顺序是困难的，所以请自行设计好样式文件（你也可以依赖同一份css文件中的顺序）

### 使用css

```
// 在模块中直接引用样式文件
// 但这有一个副作用会在dom中添加额外的`style`标签
require("./stylesheet.css")
```

## 分离出来的css打包文件

结合[extract-text-webpack-plugin](https://github.com/webpack/extract-text-webpack-plugin)就可以产出独立的css文件。

结合代码分割技术我们可以使用两种不同的方式：

 - 为每一份初始块生成一个css文件，在额外的块中内嵌样式信息（推荐）
 - 为每一份初始块生成一个css文件，并且包含其他块的css

推荐第一种方法主要是因为对于初始加载时间是最优的。在多入口文件的小型app中推荐第二种方法是因为考虑HTTP请求并发上限以及缓存。

### 插件安装

从npm中安装插件

```
npm install extract-text-webpack-plugin --save
```

### 常规用法

为了使用插件你需要改变配置，使用特殊的loader将样式输出到css文件。在代码编写后webpack优化阶段插件会检查哪个相关的模块需要被抽离（在第一种方法中只有初始块）。这些模块通过nodejs执行得到内容，另外模块被会重新编译到原先的包中代替空的模块。

为被抽离的模块创建了新的内容。

### 初始块中的样式臭历程单独的css文件

这个例子展示了多入口，但是也同样适合但入口。

```
// webpack.config.js
var ExtractTextPlugin = require("extract-text-webpack-plugin");
module.exports = {
    // The standard entry point and output config
    entry: {
        posts: "./posts",
        post: "./post",
        about: "./about"
    },
    output: {
        filename: "[name].js",
        chunkFilename: "[id].js"
    },
    module: {
        loaders: [
            // Extract css files
            {
                test: /\.css$/,
                loader: ExtractTextPlugin.extract("style-loader", "css-loader")
            },
            // Optionally extract less files
            // or any other compile-to-css language
            {
                test: /\.less$/,
                loader: ExtractTextPlugin.extract("style-loader", "css-loader!less-loader")
            }
            // You could also use other loaders the same way. I. e. the autoprefixer-loader
        ]
    },
    // Use the plugin to specify the resulting filename (and add needed behavior to the compiler)
    plugins: [
        new ExtractTextPlugin("[name].css")
    ]
}
```

### 所有的样式都被抽离成css文件

使用第二种方法你只用设置`allChunks`成`true`。

```
// ...
module.exports = {
    // ...
    plugins: [
        new ExtractTextPlugin("style.css", {
            allChunks: true
        })
    ]
}
```

### 在通用块中的样式

你可以结合CommonChunkPlugin使用独立的css文件。

```
// ...
module.exports = {
    // ...
    plugins: [
        new webpack.optimize.CommonsChunkPlugin("commons", "commons.js"),
        new ExtractTextPlugin("[name].css")
    ]
}
```
