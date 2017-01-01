> # 使用插件

使用插件添加与webpack打包有关的典型功能。举个例子，[BellOnBundlerErrorPlugin](https://github.com/senotrusov/bell-on-bundler-error-plugin)插件能在打包器工作的进程中输出错误信息。

### 内置插件

通过使用webpack的插件属性。

```
// webpack should be in the node_modules directory, install if not.
var webpack = require("webpack");

module.exports = {
    plugins: [
        new webpack.ResolverPlugin([
            new webpack.ResolverPlugin.DirectoryDescriptionFilePlugin("bower.json", ["main"])
        ], ["normal", "loader"])
    ]
};
```

### 其他插件

非内置插件可以通过npm下载或者其他途径。

```
npm install component-webpack-plugin
```

```
var ComponentPlugin = require("component-webpack-plugin");
module.exports = {
    plugins: [
        new ComponentPlugin()
    ]
}
```

当通过npm安装第三方插件时我们建议使用这个工具： https://www.npmjs.com/package/webpack-load-plugins

这会检查所有安装在依赖中的第三方插件然后在需要的时候进行懒加载。
