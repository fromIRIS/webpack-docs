> # 使用loaders

## 什么是loaders

loaders是应用在app中静态资源的转换程序。他们是一些能把资源文件中的源代码作为参数并且返回出新的源代码的函数。

举个例子：你可以使用loaders告诉webpack去加载coffeescript或者jsx

### loader 特征

 - loaders可以链式调用，他们可以应用到资源的管道系统。最后一个loader期待返回js，余下的其他loader可以返回任意形式的源代码，都会被传递到下一个loader。
 - loaders可以异步或同步。
 - loaders跑在nodejs
 - loaders接受参数。这可以用来传递配置给loaders
 - loaders可以在配置中绑定拓展后缀/正则
 - loaders可以在npm中发布下载
 - 在loader的package.json的main字段可以导出loader
 - loader能连接配置
 - 插件可以给loaders更多特征
 - loaders可以发射额外的随意文件

## 解析loaders

loaders与模块有想同的解析方式。一个loader模块预计暴露出一个函数并且使用js在nodejs中编写。通常情况下你在npm中管理loaders，但你也可以在app中管理loaders文件。

### 引用loader

按照约定（并不强制）loaders通常命名为`xxx-loader`，xxx是一名字。比如`json-loader`。

你可以通过全名引用loaders（json-loader），或者通过缩略名（json）

loader的名字的约定和搜索优先顺序由webpack配置api中的`resolveLoader.moduleTemplates`定义。

loader名字的约定是方便的，尤其是用`require()`进行loader引用的时候。看下列使用方法。

### 安装loaders

```
npm install xxx-loader --save-dev
```

## 使用方法

有许多中在app中使用loaders的方法：

 - require
 - 在配置文件中进行配置
 - 通过CLI配置

### 使用require

> 注意： 尽可能的避免使用require，如果你打算把scripts标签放入环境无关的环境。对特定的loaders使用配置约定。

用require语句指定loaders是可行的，只需要使用`!`把loaders跟资源分隔开，每一部分都相对于当前目录进行解析。

```
require("./loader!./dir/file.txt");
// => uses the file "loader.js" in the current directory to transform
//    "file.txt" in the folder "dir".

require("jade!./template.jade");
// => uses the "jade-loader" (that is installed from npm to "node_modules")
//    to transform the file "template.jade"
//    If configuration has some transforms bound to the file, they will still be applied.

require("!style!css!less!bootstrap/less/bootstrap.less");
// => the file "bootstrap.less" in the folder "less" in the "bootstrap"
//    module (that is installed from github to "node_modules") is
//    transformed by the "less-loader". The result is transformed by the
//    "css-loader" and then by the "style-loader".
//    If configuration has some transforms bound to the file, they will not be applied.
```

### 配置

你可以把loaders通过配置与正则绑定：

```
{
    module: {
        loaders: [
            { test: /\.jade$/, loader: "jade" },
            // => "jade" loader is used for ".jade" files

            { test: /\.css$/, loader: "style!css" },
            // => "style" and "css" loader is used for ".css" files
            // Alternative syntax:
            { test: /\.css$/, loaders: ["style", "css"] },
        ]
    }
}
```

### cli

你可以通过cli的拓展绑定loaders：

```
$ webpack --module-bind jade --module-bind 'css=style!css'
```

jade loader处理.jade文件，style cssloader处理.css文件

### 查询参数

loaders穿衣通过查询字符串传递查询参数（跟web一样）。查询字符串使用`？`添加到loader，类似`url-loader?mimetype=image/png`

注意：查询字符串的形式取决于loader。查询loader的文档。大多数loaders接受常规形式的参数(`?key=value&key2=value2`)或者json对象（`?{"key":"value","key2":"value2"}`）

**require**

```
require("url-loader?mimetype=image/png!./file.png");
```

**配置**

```
{ test: /\.png$/, loader: "url-loader?mimetype=image/png" }
```
或者

```
{
    test: /\.png$/,
    loader: "url-loader",
    query: { mimetype: "image/png" }
}
```


**cli**

```
webpack --module-bind "png=url-loader?mimetype=image/png"
```

