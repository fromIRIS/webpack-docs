> # code Splitting 代码分割

对于大型apps而言将所有代码都放在一个文件中是不高效的。尤其是一些代码块只要某些情况下才需要加载。webpack有一个特性可以将代码分割成按需加载的块。其他一些打包器称呼为「layers」「rollups」「fragments」。这个特性叫做「code splitting」

这是一个可选择的特性，你可以在代码中定义分割点。webpack处理依赖，导出文件以及运行时。

声明一个常见的误解：代码分割不只是提取通用代码放入可共用的块。更显著的特性是可以将代码分割成按需加载的块。这可以控制初始下载更小的代码然后当应用需要的时候下载其他代码。

### 定义分割点

AMD和commonjs定义了不同的方法按需加载代码。他们都被支持并且充当分割的点。

#### commonjs `require.ensure`

```
require.ensure(dependencies, callback)
```

`require.ensure`方法确保当回调调用的时候`dependencies`里的每个依赖都会被同步的引入。`require`作为参数传递到回调函数中。

```
require.ensure(["module-a", "module-b"], function(require) {
    var a = require("module-a");
    // ...
});
```

注意：`require.ensure`只加载模块，并不执行。

#### AMD `require`

AMD定义了一个异步的`require`方法。

```
reuqire(dependices, callback)
```

当调用的时候，所有依赖都被加载，callback函数中能得到依赖暴露出来的对象。

例子：

```
require(["module-a", "module-b"], function (a, b) {// ...})
```

note： AMD`require`加载并执行模块。在webpack中模块由左向右执行

note： 省略回调函数是允许的。


#### es6 Modules

webpack不支持es6模块，根据你的编译器创建的格式直接使用`require.ensure`或者`require`。

webpack`1.x.x`（2.0.0支持）原生不支持es6模块。但是你可以使用编译器得到，比如：babel。把es6的import语法编译成commonjs和amd模块。这个方法是有效的但是也有一个重要的动态加载警告。

模块额外的语法（`import x from 'foo'`）是特意被设计为可静态分析的。这就意味着你不能动态的import。

```

// 这是非法的
['lodash', 'backbone'].forEach(function (item) {import item})
```

幸运的是，有个js api 「loader」用来处理动态使用例子：`System.load`(或者)`System.import`。这个API跟require变量的作用一样。然而多数编译器不支持转变`System.load`调用`require.ensure`。所以你想使用代码分割你可以直接使用require。

```
//static imports
import _ from 'lodash'

// dynamic imports
require.ensure([], function(require) {
  let contacts = require('./contacts')
})
```

#### 块内容

所有在代码分割点引用的依赖会进入新的块中，其中的依赖也会递归增加。

如果传入了一个函数表达式（或者绑定了一个函数表达式）作为分割点的回调函数。webpack会将所有依赖都放到块中。

#### 块chunk优化

如果两个块包含相同的模块，他们会被合并成一个块。

#### 块chunk加载

根据配置选项`target`一个用于块加载的运行时会被添加到打包文件`bundle`中。举个例子，将target设置成`web`块会通过jsonp被加载。一个块只会被加载一次，并且平行的请求会被合并成一个。运行时会检查已经加载的块是否满足其他块。

#### 块的类型

**入口块**

一个入口块包含了运行时外加一系列的模块。如果块包含了模块0，运行时会执行它。如果没有运行时会等待包含模块0的块然后执行。

**普通的块**

普通块不包含运行时，这只包含一系列的模块。结构取决于块加载的算法。举个例子，jsonp中模块会被包装在jsonp的回调函数中。块也会包含一系列满足的id。

**初始块（不是入口）**

初始的块是个普通的块。唯一的区别在于优化机制认为初始块更重要因为初始块计算初始的时间。这种块的类型会出现在`commonsChunkPlugin`插件的结合中。

**分割app和vendor代码**

把你的app分割成两个文件，叫做app.js和vendor.js。你可以在vendor.js中依赖vendor类型的文件，然后传递这些名字到`commonsChunkPlugin`中。

```
var webpack = require("webpack");

module.exports = {
  entry: {
    app: "./app.js",
    vendor: ["jquery", "underscore", ...],
  },
  output: {
    filename: "bundle.js"
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin(/* chunkName= */"vendor", /* filename= */"vendor.bundle.js")
  ]
};
```

这会从app块中移除所有vendor块。bundle.js将会保留app的代码，没有任何依赖。这些移除的代码将会留在vendor.bundle.js中。在你的html中加载

```
<script src="vendor.bundle.js"></script>
<script src="bundle.js"></script>
```

### 多入口的块

设置多入口的点产出多入口的块是可行的。入口块包含一个运行时且当前页面只有一个运行时。（当然也有例外）

#### 运行多个入口点

使用`commonChunkPlugin`插件运行时会被移到通用的块中。入口点现在在初始的块中。然而只有一个初始块会被加载，多个入口块会被加载。这就显示了在单页面执行多个入口点的可能性。

```
var webpack = require("webpack");
module.exports = {
    entry: { a: "./a", b: "./b" },
    output: { filename: "[name].js" },
    plugins: [ new webpack.optimize.CommonsChunkPlugin("init.js") ]
}
```

```
<script src="init.js"></script>
<script src="a.js"></script>
<script src="b.js"></script>
```

#### 通用的块

`CommonsChunkPlugin`会把多入口的块移到一个新的入口块（通用块），运行时也会被移到通用的块。这意味着老的入口块现在变成了初始块。

#### 优化

以下优化插件可以根据特定条件合并块。

```
LimitChunkCountPlugin
MinChunkSizePlugin
AggressiveMergingPlugin
```

#### 命名块

require.ensure函数接受额外的第三个参数。这个参数一定是一个字符串。如果两个分割点传递了相同的字符串他们会使用相同的块。

`require.include`

require.include是一个webpack特定的函数用来给当前块添加一个模块，但是不执行。（表达式会从bundle中移除。）

例子：

```
require.ensure(["./file"], function(require) {
  require("./file2");
});

// is equal to

require.ensure([], function(require) {
  require.include("./file");
  require("./file2");
});
```

如果在多子块的情况下require.include是好用的，require.include在父块中会引入模块，在子块中该模块的实例会消失。

例子： 


- [simple](https://github.com/webpack/webpack/tree/master/examples/code-splitting)
- [with bundle-loader](https://github.com/webpack/webpack/tree/master/examples/code-splitting-bundle-loader)
- [with context](https://github.com/webpack/webpack/tree/master/examples/code-splitted-require.context)
- [with amd and context](https://github.com/webpack/webpack/tree/master/examples/code-splitted-require.context-amd)
- [with deduplication](https://github.com/webpack/webpack/tree/master/examples/code-splitted-dedupe)
- [named-chunks](https://github.com/webpack/webpack/tree/master/examples/named-chunks)
- [multiple entry chunks](https://github.com/webpack/webpack/tree/master/examples/multiple-entry-points)
- [multiple commons chunks](https://github.com/webpack/webpack/tree/master/examples/multiple-commons-chunks)

[可执行的demo](http://webpack.github.io/example-app/)可以在devTools中查看网络

