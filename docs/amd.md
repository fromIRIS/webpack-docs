> # AMD

AMD（异步模块系统）为了适应那些觉得commonjs模块系统还没在browser准备好的人产出的，因为他们觉得commonjs的本质是同步的。

AMD提出了现代js的标准，以至于模块能异步的加载依赖，解决了同步加载的问题。

### 说明

模块有defined函数来定义

`define`

define函数用于使用AMD定义一个模块。这只是一个带有签名的函数。

```
define(id?: String, dependencies?: String[], factory: Function|Object);
```

`id`

指定了模块的名字，可选。

`dependencies`

这个参数定义了被定义的模块所依赖的模块。是一个包含了模块标识符的数组，可选项。但是如果省略，默认设置成[“require”, “exports”, “module”].

`factory`

最后一个参数定义了模块内容，可以是个函数（立马执行），或者对象。如果factory是个函数，返回值会变成模块的导出值。

### 例子

#### 命名模块

定义一个名为myModule依赖jQuery的模块

```
define('myModule', ['jquery'], function($) {
    // $ is the export of the jquery module.
    $('body').text('hello world');
});
// and use it
require(['myModule'], function(myModule) {});
```

注意：在webpack中命名模块只在本地使用，在require.js中命名模块是全局的。

#### 匿名模块

```
define(['jquery'], function($) {
    $('body').text('hello world');
});
```

#### 多依赖

```
define(['jquery', './math.js'], function($, math) {
    // $ and math are the exports of the jquery module.
    $('body').text('hello world');
});
```

#### 导出

```
define(['jquery'], function($) {

    var HelloWorldize = function(selector){
        $(selector).text('hello world');
    };

    return HelloWorldize;
});
```

