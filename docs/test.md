> # 测试

有两种方式可以测试web应用：

 - 浏览器：你可以得到更现实的测试，但是你需要准备更多的基础建设，且测试需要花费更多时间。你可以测试dom。
 - nodejs：你不能测试dom，但是测试会更快。

### 浏览器测试

#### mocha-loader

mocha-loader使用mocha框架执行你的代码。如果执行代码你会在浏览器看到测试结果。

提示：当在bash命令行使用`！`时，你需要使用`\`转义。

```
webpack 'mocha!./test.js' testBundle.js
<!--index.html is a HTML page which loads testBundle.js-->
open index.html
```

#### webpack-dev-server

webpack-dev-server会自动的创建加载脚本的HTML页面。当文件改变时会重新执行。

```
webpack-dev-server 'mocha!./test.js' --hot --inline --output-filename test.js
open http://localhost:8080/test
```

提示：使用`--hot`服务器只会在该文件或该文件的依赖有变化时重新执行测试。

#### karma与webpack

你可以将webpack与karma一起使用，将webpack作为[预处理器](https://github.com/webpack/karma-webpack)加进karma的配置中

### nodejs


