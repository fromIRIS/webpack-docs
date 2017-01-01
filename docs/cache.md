> # 长期缓存

为了有效的缓存文件，文件需要带有hash或者版本号的URL。你可以人为的修改产出的文件的版本号`v.1.3`但是这样不是很方便。额外的人工才操作以及没有改变的文件不从缓存中加载。

webpack可以根据文件名给文件添加hash值。处理文件的loaders（worker-loader，file-loader）早已经实现，对于chunks你需要开启它，有两种等级：


 1. 计算所有chunks的hash
 2. 为每个chunk计算hash

### 选择1：一个bundle一个hash

选择1通过为文件名配置hash选项来开启

`webpack ./entry output/[hash].bundle.js`

```
{
    output: {
        path: path.join(__dirname, "assets", "[hash]"),
        publicPath: "assets/[hash]/",
        filename: "output.[hash].bundle.js",
        chunkFilename: "[id].[hash].bundle.js"
    }
}
```

### 选择2： 每个chunk一个hash

选项2通过添加[chunkhash]到chunkFilename配置选项来开启。

`--output-chunk-file [chunkhash].js`

```
output: { chunkFilename: "[chunkhash].bundle.js" }
```

记得你需要在html中引用带有hash的入口chunk。你可能想从stats提取hash或者文件名。结合热替换你必须使用选项1，但不是在`publicPath`配置项中。

### 从stats中获取文件名

你可能想得到最后的文件名字嵌入你的html中。这个信息在webpack的stats中是可以获取的。如果你是使用CLI你可以运行`--json`去得到以json形式输出的stats。

你可以在配置文件中加入[assets-webpack-plugin](https://www.npmjs.com/package/assets-webpack-plugin)插件来允许你得到stats对象。接下来是一个将此写入文件的例子。

```
plugins: [
  function() {
    this.plugin("done", function(stats) {
      require("fs").writeFileSync(
        path.join(__dirname, "..", "stats.json"),
        JSON.stringify(stats.toJson()));
    });
  }
]
```

stats json包含了好用的对象--`assetsByChunkName`，这个对象包含了以chunk为键名文件名为键值的对象。

