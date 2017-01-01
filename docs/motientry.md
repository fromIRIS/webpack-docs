> # 多入口的点

要求：[代码分割](http://webpack.github.io/docs/code-splitting.html)

如果你需要多个打包文件给多个html页面使用，你可以使用「多入口点」特性。这会同时生成多个打包文件。额外的chunks可以被这些入口chunks共享，模块只会被构建一次。

> 提示： 如果你想从模块中开始一个入口chunk，这是个错误的想法。使用代码分割！

每一个入口chunk都包含了webpack运行时，所以你只用在每个页面加载一个入口chunk（提示：可以使用commonsChunkPlugin插件去绕过这个限制将运行时放入单个chunk中。）

### 配置

为了使用多入口点你可以往entry选项中传入一个对象。键名代表了入口点的名字，键值代表了入口点。

当应用多入口点时必须改写默认的`output.filename`选项。不然每个入口点都会写入相同的文件。使用`[name]`得到入口点的名字。

### 最简单的配置例子

```
{
    entry: {
        a: "./a",
        b: "./b",
        c: ["./c", "./d"]
    },
    output: {
        path: path.join(__dirname, "dist"),
        filename: "[name].entry.js"
    }
}
```

### 例子

 - [multiple-entry-points](https://github.com/webpack/webpack/tree/master/examples/multiple-entry-points)
 - [multi-part-library](https://github.com/webpack/webpack/tree/master/examples/multi-part-library)
 - [multiple-commons-chunks](https://github.com/webpack/webpack/tree/master/examples/multiple-commons-chunks)