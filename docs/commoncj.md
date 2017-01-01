> # commonjs

CommonJS团队通过确保每个模块都在自己的命名空间中执行定义了一种解决javascript作用域问题的模块形式。

commonjs通过强制模块输出正确的想要暴露在全局的变量，同时定义好正确工作所需的其他模块。

为了实现这些`commonjs`提供了两个工具：
 
 - require()函数，允许在当前作用域带入模块。
 - module对象，允许从当前作用域输出一些东西。

必须来个`hello world`例子：

### 纯javascript

这有个不使用commonjs的例子：

在salute.js文件中定义

```
var MySalute = "Hello";
```

然后在第二个文件world.js中取值

```
console.log(MySalute) // hello
```

### 模块定义

实际上，MySalute因为没有定义world.js不会正常工作，我们需要把每个script定义成模块。

```
// salute.js
var MySalute = "Hello";
module.exports = MySalute;
```

```
// world.js
var Result = MySalute + "world!";
module.exports = Result;
```

这里我们使用了特殊的对象module然后把变量引用赋值给了module.exports，所以commonjs模块系统知道这事我们想要暴露给世界的模块内的对象。salute.js暴露了MySalute，world.js暴露了Result

### 模块依赖

我们离成功就差了一步：依赖定义。我们早就把每个script定义成了独立的模块，但是world.js还需要知道谁定义了MySalute

```
// world.js
var MySalute = require("./salute");
var Result = MySalute + "world!";
module.exports = Result;
```