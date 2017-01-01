> # 热加载

注意模块热替换机制目前还处于试验阶段。

## 介绍

模块热替换（HMR）会在应用运行时替换、增加、移除模块而不用重新加载页面。

### 要求

 - 使用plugins http://webpack.github.io/docs/using-plugins.html
 - 代码分割： http://webpack.github.io/docs/code-splitting.html
 - webpack-dev-serve http://webpack.github.io/docs/webpack-dev-server.html

### HMR如何工作

webpack在构建打包的过程中增加一个小型的HMR运行时到打包文件中。当构建完成时，webpack不会退出并保持活跃。观察源文件的变化。如果webpack检测到文件的变化，他会重新构建变化的模块。取决于webpack的设置，webpack会给HMR运行时发送一个信号，或者HMR运行时会查询webpack的变化。不管哪种形式，改变的模块会被发送到HMR运行时，然后应用热更新。首先HMR运行时会检查更新的模块能不能自我accept，如果不会那么会检查依赖这些变化模块的模块。如果这些模块也不会accept，则会冒泡到下一个层级，知道能执行accept方法，或者到达app的入口文件，这也意味着热更新是失败的。

### 从app的视角

app代码请求HMR检查更新，HMR运行时异步的下载更新然后告知app代码更新是可用的。然后app代码告知HMR运行时应用这些更新，然后HMR同步的应用更新。app代码要不要依赖用户的输入取决于开发者自己。

### 从编译器（webpack）的视角

除了普通的资源编译器还需要触发「update」以允许从之前的版本更新到当前版本，「update」包含两部分：

1. 更新清单（json）
2. 一个或多个更新的chunk（js）

清单包含新的hash值和更新的chunk。

更新的chunks包含了所有模块。

编译器额外确保模块和chunk的id在build之间是不变的。他使用「record」json文件去存储或者在内存中存储。

### 从模块视角

HMR是一个可选的特性，所以这只会影响包含HMR代码的模块。文档描述的API在模块中都是可用的。通常情况下模块开发者需要写一个handles，这个handles在这个模块的依赖更新时会被触发。他也能写一个当此模块更新时就会触发的handle。在大多数情况下在每个模块中写HMR的代码不是强制的。如果一个模块没有HMRhandles那么更新会传递到下一个层级。这就意味了一个handle能沿着模块树处理一个更新。如果一个模块更新了，整个模块树都会重载（只重载不转移）

### 从HMR运行时视角

对于模块系统来说运行时是额外的代码用来触发追踪父模块和子模块。

在管理方面运行时支持两个方法： `check`和`apply`

`check`从更新清单发起http请求，如果请求失败则没有更新的内容。不然请求来的chunks列表会跟当前已经加载的chunks进行对比。每一个已加载的chunk与之对应的更新chunk会被下载。所有的模块更新在更新的同时都会存到运行时中。此时runtime进入「ready」状态，意味着更新已经下载等待被应用。

在ready状态下每个新的chunk请求也会被下载。

`apply`方法标记所有更新的模块为无效。对于每一个无效的模块需要一个updatehandle或者在其父模块上需要update handle。不然无效打包文件会将所有父级都标记为无效。这个过程一直持续到没有冒泡发生。如果冒泡到入口chunk则热替换失败。

现在所有无效模块都被处理了但是没有加载。然后当前的hash更新所有的accept被调用。runtime重新回到「idle」状态。

### 生成文件

### 我能用它做什么

你可以在开发环境当成livereload去使用它。事实上webpack-dev-server支持热替换模式，尝试在重新加载整个页面之前用HMR替换。你只需要添加`webpack/hot/dev-server`入口点然后用`--hot`开启开发服务器。

`webpack/hot/dev-server`当HMR更细失败后会重新加载整个页面。如果你想用你自己的方式重载页面，你可以添加在入口点`webpack/hot/only-dev-server`。

你也当做更新机制在生产环境使用。你需要写相关的代码与HMR交互。

一些loaders生产的模块已经是可以热更新的。`style-loader`能改变样式表。你不需要做其他特殊的东西。

### 使用它需要做什么

一个模块只有你accept才会更新。所以你需要写`module.hot.accept`在模块的父级及以上。举例：router是个好地方。

如果你只是想与webpack-dev-server一起使用，只用加`webpack/hot/dev-server`当做入口点。不然你需要使用能调用`check`和`apply`HMR代码。

你需要开启编译器的record去跟踪编译过程中的id（watch方式和webpack-dev-server会将records保存到内存，所以在开发过程中不需要）

你需要在编译器开启HMR并且添加HMR运行时。

### 为什么看起来这么酷

- 这是模块层级的实时更新
- 可以在生产环境使用
- 更新考虑代码分割，只会下载app所需部分的更新。
- 你可以在你的app部分代码使用这个功能，并不影响其他模块。
- 如果HMR关闭了，所有HMR代码会被移除（在`if(module.not)`包裹下）


## 练习

配合webpack使用热替换你需要4件事：
 - records （--records-path, recordsPath: ...）
 - 全局允许热替换（HotModuleReplacementPlugin）
 - 在你的代码里加入热替换`module.hot.accept`
 - 热替换管理代码`module.hot.check, module.hot.apply`

小栗子：

```
/* style.css */
body {
    background: red;
}
```

```
/* entry.js */
require("./style.css");
document.write("<input type='text' />");
```

然后就可以使用dev-server使用热替换

```
npm install webpack webpack-dev-server -g
npm install webpack css-loader style-loader
webpack-dev-server ./entry --hot --inline --module-bind "css=style\!css"
```

dev-server提供内存records，这对于开发是很好地。

`--hot`开关开启代码热替换。

> 这会添加HotModuleReplacementPlugin，确保不么添加`--hot`，不么就在webpack.config.js里添加HotModuleReplacementPlugin，但是永远不要同一时间两个都加。HMR插件会添加两次。

有个特殊的控制代码`webpack/hot/dev-server`，会被`--inline`自动添加。（你不用手动添加在webpack.config.js）

`style-loader`已经包含热替换代码。

阅读更多关于如何写热替换代码[hot module replacement](http://webpack.github.io/docs/hot-module-replacement.html)