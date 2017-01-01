> # 打包性能

如果你在寻找加速webpack打包的方法，你可能要通过以下几种方法去更加深入的提高你配置的webpack的打包性能。

## 逐步的打包

确保每次打包的时候不会全部重新打包。webpack有一个强大的缓存层允许你使用内存中早已编译好的模块，以下几种方法帮助使用：

 - webpack-dev-server： 将所有资源存到内存，性能最好。
 - webpack-dev-middleware：与webpack-dev-server有相同的性能，提供给有深层定制的用户
 - webpack --watch 或者 `watch: true` 有缓存，但是缓存到硬盘，性能一般。

## 不解析某些模块

使用`noParse`可以在解析时排除大的第三方库，但是会中断。

## 打包过程的信息

有个[分析工具](http://webpack.github.io/analyse/)可以提供详细的分析和一些可以帮助你优化打包文件大小和性能的有用信息。

## chunks

从内部表现生成源文件代价是高的。只有当这个chunk内部没有任何改变时，chunk都由自己缓存。大多数chunk取决于自身包含的模块，但是入口chunk不同，如果额外的chunk名字改变了，入口块同样会被认为是脏的，即被修改过的。所以使用在文件名中使用`[hash]或者[chunkhash]`时，入口chunk几乎会在每次修改中都重新构建。

使用HMR入口chunk会嵌入编译的hash所以每次改变也会被认为是脏的。

## sourcemaps

优秀的sourceMaps会减慢build

`devtool: "source-map"` 不会缓存模块的sourcemaps而且会重新生成chunk的sourcemaps。这是给生产环境用的。

`devtool: "eval-source-map"` 跟上一种异样好，但是会缓存模块的sourcemaps，对于重复构建速度会快。

`devtool: "eval-cheap-module-source-map"` 只提供行的sourcemaps，速度更快

`devtool: "eval-cheap-source-map"` 不会生成模块的sourcemaps，例如jsx向js的mapping

`devtool: "eval"` 性能最好，但只生产模块的sourcemaps，在一些情况下这是足够的。使用`output.pathinfo: true`编译

UglifyJsPlugin插件使用sourcemaps生成对应源代码的错误信息，但是sourcemaps是慢的，在生产环境使用是ok的，如果构建时速度太慢（甚至不能完成），你需要关闭这个功能。`new UglifyJsPlugin({ sourceMap: false })`

## RESOLVE.ROOT 对比 RESOLVE.MODULESDIRECTORIES

只在嵌套路径下使用[resolve.modulesDirectories](http://webpack.github.io/docs/configuration.html#resolve-modulesdirectories)，大多数路径使用[resolve.root](http://webpack.github.io/docs/configuration.html#resolve-root)，这可以给出[性能展示](https://github.com/webpack/webpack/issues/1574#issuecomment-157520561) 看[讨论](https://github.com/webpack/webpack/issues/472#issuecomment-55706013)


## 优化的插件

只在生产环境使用优化用的插件

## 提前取得模块

[prefetch](http://webpack.github.io/docs/list-of-plugins.html#prefetchplugin)

## 动态链接的库

如果你有大量很少改变的模块（比如说vendor），chunking不会带来多大的性能提升（commonChunkPlugin），这有两个插件可以在分隔的大包进程中创建一个打包文件，但是也会在appbundle中引用。

提前创建DLL包你需要使用Dllplugin，这是[例子](https://github.com/webpack/webpack/tree/master/examples/dll)。这会触发公共的打包文件和私有的清单文件。

从APP打包文件中引用DLL打包文件，你需要使用`DllRefencePlugin`这是[例子](https://github.com/webpack/webpack/tree/master/examples/dll-user)，在找到Dll打包文件之前会阻止依赖app的文件。