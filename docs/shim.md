> # 垫板模块（shim）

不是所有js文件都可以直接使用webpack。此文件可能是webpack不支持的文件，甚至没有任何模块形式。

webpack提供一些loaders使这些文件可以跟webpack一起工作。

下面的例子使用require保持简洁。你通常都会在webpack中配置他们。

## 输入

如果一个文件的依赖不是通过require()引入的，你可以使用以下loader中的一种。

### `imports-loader`

import loader允许你根据不同的全局变量去使用模块。

对于依赖像$或者this的第三方模块这是很方便的。imports loader会添加必要的`require('whatever')`调用。所以这些模块可以跟webpack工作。

例子： 

file.js 需要一个全局的$变量，你也有一个应该被使用的jquery模块。

`require("imports?$=jquery!./file.js")`

file.js需要一个全局的配置变量`xConfig`，你希望是`{value: 123}`

`require("imports?xConfig=>{value:123}!./file.js")`

file.js需要一个全局的this对象。

`require("imports?this=>window!./file.js") or require("imports?this=>global!./file.js")`

### [plugin](http://webpack.github.io/docs/list-of-plugins.html) 提供插件

这个插件使得一个模块在任何模块中能称为一个变量。只有当你使用这个变量的时候才会依赖这个模块。

例子： 不需要写`require("jquery")`就能在任何模块中使用$和jquery变量。

```
new webpack.ProvidePlugin({
    $: "jquery",
    jQuery: "jquery",
    "window.jQuery": "jquery"
})
```

## 输出

不暴露值的文件。

### `exports-loader`

这个loader暴露文件的内部变量。

例子：

这个文件在全局上下文定义了一个变量`var XModule = ...`

`var XModule = require("exports?XModule!./file.js")`

这个文件在全局上下文定义了多个变量 `var XParser, Minimizer`

`var XModule = require("exports?Parser=XParser&Minimizer!./file.js"); XModule.Parser; XModule.Minimizer`

这个文件设置了一个全局变量 `XModule = ....`

```
require("imports?XModule=>undefined!exports?XModule!./file.js") (import to not leak to the global context)
```

这个文件在window对象下设置了属性 `window.XModule = ...`

```
require("imports?window=>{}!exports?window.XModule!./file.js")
```

## 修复错误使用的模块风格

有些模块使用了错误的模块风格。你想去修复并且告诉webpack不要使用这种风格。

### 使模块风格失效

例子：

#### AMD失效

require("imports?define=>false!./file.js")

#### CommonJs失效

require("imports?require=>false!./file.js")

### [配置](http://webpack.github.io/docs/configuration.html) 选项 `module.noParse`

webpack会使解析失效，因此你不能使用依赖，这对已经打包好的第三方库比较实用。

例子： 

```
{
    module: {
        noParse: [
            /XModule[\\\/]file\.js$/,
            path.join(__dirname, "web_modules", "XModule2")
        ]
    }
}
```

> exports 和 module任然是可用的，你可以使用imports-loader使他们失效。

### `script-loader`

这个loader在全局上下文中评估代码，就跟你在script中添加代码一样。这种方式每一个第三方库都能正常工作，require、module等都会失效。

> 注意：此文件会被当做字符串加入到bundle中，不会被webpack压缩，所以我们需要使用压缩版本。也没有作用于这种loader加载的第三方库的开发者工具。

## 暴露

有些情况你想一个模块暴露出自己。

除非必须不然少用（providePlugin更好）

### `expose-loader`

这个loader将模块暴露到全局上下文中。

例子： 

将file.js暴露到全局上下文中的XModule变量。

`require("expose?XModule!./file.js")`

另一个例子：

```
   require('expose?$!expose?jQuery!jquery');

   ...

   $(document).ready(function() {
   console.log("hey");
   })
```

通过jquery文件暴露到全局上下文，你可以在项目中的任何地方使用jquery。同理你想使用bootstrap也可以通过这种方法。

注意： 使用太多全局变量会使你的app缺少效率，如果你想使用大量命名空间，考虑在项目中加入[Babel runtime](http://babeljs.io/docs/plugins/transform-runtime/)

## loader的顺序

在非常小的应用场景下你需要应用不只一个配置，你需要使用正确的loader顺序。内嵌：`expose!imports!exports` ，配置项：expose before imports before exports.。
