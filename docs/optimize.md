> # 优化

### 最小化

去最小化你的脚本（如果你使用css-loader还有css），webpack支持一个简单的选项：

`--optimize-minize`或者 `new webpack.optimize.UglifyJsPlugin()`

这是个最简单但是最有效的优化你的webapp的方法。

正如你所知道的（如果你有持续阅读文档）webpack会给模块和块`id`去标识他们。webpack可以改变ids的分配去得到最短的id长度作用于常用的ids：

`--optimize-occurence-order`或者`new webpack.optimize.OccurenceOrderPlugin()`

入口块对于文件大小有更高的优先级。

### 删除重复数据

如果你使用一些有很多依赖的第三方库，这可能会发生一些文件会相同。webpack会发现这些文件并删除他们。这会防止你的包里包含相同的代码，相反的会在运行时应用一个函数的引用。这不影响语义，你可以这样开启：

`--optimize-dedupe`或者`new webpack.optimize.DedupePlugin()`

这个特性在入口文件中添加了一些开销。

### 块

在书写代码的时候，你可能早已经添加了许多代码分割点来按需加载代码。当编译后你可能会注意到这么多的块对于http的开销来说是还是太小体量的。幸运的是，webpack可以通过后处理的方式去合并他们。你可以提供两种选项：

 - 限制块的最大数量`--optimize-max-chunks 15 new webpack.optimize.LimitChunkCountPlugin({maxChunks: 15})` 
 - 限制块的最小尺寸 `--optimize-min-chunk-size 10000 new webpack.optimize.MinChunkSizePlugin({minChunkSize: 10000})`

webpack通过合并chunk来解决这个优化问题（webpack更倾向于合并有相同模块的chunk）。没有东西会被合并到入口chunk，所以不会影响页面加载时间。

### 单页应用

webpack就是被设计优化像单页应用这种web应用的。

你可以将app中的代码分割成多个chunk，由路由判断来加载。入口chunk只包含路由和一些第三方资源，没有内容。当你的用户在app中操作时这会工作顺利，但是对于不同路由的初始加载你可能需要一个来回：第一步获取路由第二步获取当前页内容。

如果你使用HTML5 history的API跳转当前页面，通过客户端代码服务器能知道哪个页面被请求。为了节省桐乡服务器的来回路程你可以包含内容块到返回中。直接添加script标签是可行的。浏览器会加载平行的chunks。

```
<script src="entry-chunk.js" type="text/javascript" charset="utf-8"></script>
<script src="3.chunk.js" type="text/javascript" charset="utf-8"></script>
```

你可以从stats中提取文件名（[stats-webpack-plugin](https://www.npmjs.com/package/stats-webpack-plugin)能用来导出构建后的stats）

### 多页应用

当你编译多页面的app，你想在多页面之间共享相同的代码。事实上结合webpack这非常容易：只需要结合多个入口点进行编译：

```
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        p3: "./page3"
    },
    output: {
        filename: "[name].entry.chunk.js"
    }
}
```

这会生成多个入口chunk：`p1.entry.chunk.js`,`p2.entry.chunk.js`和`p3.entry.chunk.js`但是其他的chunk能通过他们分享。

如果你的入口chunks有一些相同的模块，这有个很好用的插件。`CommonsChunkPlugin`识别相同的模块然后将他们放入一个通用chunk。你需要在页面中加入两个script标签。一个是通用的chunk，一个是入口chunk。

```
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        p3: "./page3"
    },
    output: {
        filename: "[name].entry.chunk.js"
    },
    plugins: [
        new CommonsChunkPlugin("commons.chunk.js")
    ]
}
```

这会形成多个入口chunk`p1.entry.chunk.js`,`p2.entry.chunk.js`和`p3.entry.chunk.js`。加上一个`common.chunk.js`，首先加载`common.chunk.js`然后加载`xx.entry.chunk.js`

你也可以通过选择入口chunks形成多个通用chunks，你也可以嵌套通用chunks

```
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        p3: "./page3",
        ap1: "./admin/page1",
        ap2: "./admin/page2"
    },
    output: {
        filename: "[name].js"
    },
    plugins: [
        new CommonsChunkPlugin("admin-commons.js", ["ap1", "ap2"]),
        new CommonsChunkPlugin("commons.js", ["p1", "p2", "admin-commons.js"])
    ]
};
// <script>s required:
// page1.html: commons.js, p1.js
// page2.html: commons.js, p2.js
// page3.html: p3.js
// admin-page1.html: commons.js, admin-commons.js, ap1.js
// admin-page2.html: commons.js, admin-commons.js, ap2.js
```

高级用法： 你可以在通用chunk中运行代码

```
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        commons: "./entry-for-the-commons-chunk"
    },
    plugins: [
        new CommonsChunkPlugin("commons", "commons.js")
    ]
};
```

看[multiple-entry-points example](https://github.com/webpack/webpack/tree/master/examples/multiple-entry-points)和[advanced multiple-commons-chunks example](https://github.com/webpack/webpack/tree/master/examples/multiple-commons-chunks)

