> #动机

现在的网站都向webapp进化：

 - 在页面中有越来越多的js
 - 现在浏览器能做更多的事情
 - 更少的页面重载刷新 在页面中有更多的代码

结论是在浏览器端存在大量的代码。

大段的代码需要被组织，模块系统提供了这个机会把代码分割成模块。

## 模块系统的风格

有许多中引入依赖导出值的标准：

 - `<script>`标签风格（没有模块系统）
 - commonjs
 - AMD及其相似
 - ES6
 - 。。。

### `<script>`标签风格

如果没有使用模块系统你将会按如下方式处理模块化的代码

```
<script src="module1.js"></script>
<script src="module2.js"></script>
<script src="libraryA.js"></script>
<script src="module3.js"></script>
```

各个模块把接口暴露给全局对象，比如`window`对象。各个模块之间可以通过全局对象进行访问互相依赖的接口。

普遍的问题：

 - 全局对象的冲突
 - 加载的顺序是重要的
 - 开发者需要解决模块的依赖问题
 - 在大项目中模块引入的数目将会非常长并且难以维护

### CommonJs：同步的`require`

这种风格使用同步的`require`方法来加载依赖和返回暴露的接口。一个模块可以通过给`exports`对象添加属性，或者设置`module.exports`的值来描述暴露对象。

```
require("module");
require("../file.js");
exports.doStuff = function() {};
module.exports = someValue;
```

CommonJs规范也在服务端nodejs中使用。

利：

 - 服务端代码可以被重用
 - npm中有大量模块
 - 简易使用

弊：

 - 阻塞调用在网络中无法应用，网络请求是异步的
 - 不能同时引入多个模块

### AMD：异步的`require`

`异步的模块定义`

其他一些模块系统（针对浏览器）对同步的`require`有问题，引出了异步的版本（一种定义模块暴露值的方式）

```
require(["module", "../file"], function(module, file) { /* ... */ });
define("mymodule", ["dep1", "dep2"], function(d1, d2) {
  return someExportedValue;
});
```

利：

 - 符合了在网络中异步请求的风格
 - 可以同时加载多个模块

弊：

 - 编码成本 难读难写
 - 看起来像是一种权宜之计

实践：

 - require.js
 - curl

### ES6 模块

ecmascript6添加了一些语言结构。

```
import "jquery";
export function doStuff() {}
module "localModule" {}
```

利：
 
 - 静态分析方便
 - 作为es的标准未来是有保证的。

弊：

 - 本地浏览器支持还需要时间
 - 这种风格几乎没有成型的模块。

### 无偏见的解决方案

开发者可以选择自己的模块风格，确保现存的代码和包能工作。添加习惯的模块风格是方便的。

## 传输

模块要在浏览器端运行，所以他们必须要从后端传输到浏览器端。

传输模块会出现两个极端：

 - 每个模块是一个请求
 - 一个请求包含了所有模块

这两种使用方法都是疯狂的，但并不理想：

 - 一个模块对应一个请求

  - 利：只有需要的模块被传输
  - 弊：多请求意味着更多的开销
  - 弊：因为请求延迟会降低应用初始化时间。

 - 一个请求包含所有模块

  - 利：更少的请求开销，更少的延迟
  - 弊：不需要的模块也会被传输

### 分块的传输

一个更灵活的传输方式会更好，两个极端方式的折中方案在大多数例子中会更好。

当编译所有模块时，把一系列模块分割成许多更小的块。

这就允许更多更小更快的请求，初始阶段不需要的模块可以通过命令加载，这样就可以加速初始加载，当你实际需要代码时也能加载相应的代码块。

「分割点」取决于开发者

[阅读更多](http://webpack.github.io/docs/code-splitting.html)

## 为什么只有javascript可以模块化？

为什么模块系统只能帮助开发者解决javascript的模块问题？还有许多其他的资源需要处理：

 - 样式stylesheets
 - 图片images
 - 字体webfonts
 - html模板
 - 。。

编译：

 - coffeescript > javascript
 - elm > javascript
 - less > css
 - jade templates > 生成html的js

使用起来应该跟下列一样方便：

```
require("./style.less");
require("./template.jade");
require("./image.png");
```

[使用loaders](http://webpack.github.io/docs/using-loaders.html)
[loaders](http://webpack.github.io/docs/loaders.html)

## 静态分析

当编译这些模块时，静态分析尝试寻找模块的所有依赖。

传统上静态分析寻找只能填写字符串（不带变量），但是`require('./template/' + templateName + '.jade')`是很普遍的结构。

许多第三方库都有不同的书写风格，一些非常诡异。

### 对策

一个聪明强大的解析器允许几乎所有现存的代码运行，但是开发者写了一些奇怪的代码，这需要找到最合适的解决方法。