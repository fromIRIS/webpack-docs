> # 第三方库和拓展

你开发了一个第三方库然后要分到编译/打包后的版本（除了模块化的版本）。你想要允许用户在script标签或者amd加载器中使用。或者你想取决于不同的预编译器而不限制用户，把这个模块作为普通的commonjs模块。

### 配置选项

webpack有三个跟这个情况相关的配置选项：`output.library`,`output.libraryTarget`,`externals`

`output.libraryTarget`允许你控制输出的类型，举例：commonjs,amd,script中使用。

`output.library`允许你指定第三方库的名字。

`externals`允许你指定第三方库的不需要经过webpack处理的依赖。但是是输出文件的依赖。这也表明了他们是在运行时环境中输入的。

### 例子

编译在script中使用的第三方库。

 - 依赖jquery，但是jquery不应该包含在打包文件中。
 - library应该在全局上下文中的Foo中可访问。

```
var jQuery = require("jquery");
var math = require("math-library");

function Foo() {}

// ...

module.exports = Foo;
```

推荐的配置（与之相关）

```
{
    output: {
        // export itself to a global var
        libraryTarget: "var",
        // name of the global var: "Foo"
        library: "Foo"
    },
    externals: {
        // require("jquery") is external and available
        //  on the global var jQuery
        "jquery": "jQuery"
    }
}
```

打包结果

```
var Foo = (/* ... webpack bootstrap ... */
{
    0: function(...) {
        var jQuery = require(1);
        /* ... */
    },
    1: function(...) {
        module.exports = jQuery;
    },
    /* ... */
});

```

### 应用以及外部资源

你也可以使用`externals`选项向应用导出一个存在的接口。举个例子，你想使用cdn资源script引用的jquery，但又明确声明通过`require('jquery')`作为依赖，你可以像这样把他指定为外部资源：`{externals: {jquery: "jQuery"}}`

### 分解以及外部资源

外部资源在分解请求之前执行，这意味着你需要指定没有分解的请求。externals中不能应用loaders，所以你需要用loader具体化一个请求。`require("bundle!jquery")  { externals: {"bundle!jquery": "bundledJQuery"}}`

