> # 什么是webpack

**webpack是一个模块打包工具**

webpack把有依赖的模块产出代表这些模块的静态资源。

### 为什么要用模块打包器？

现存的模块打包工具不适用大型项目（大型的单页应用）。开发新的模块打包工具最令人激动的原因就是[代码分割](http://webpack.github.io/docs/code-splitting.html)，并且静态资源跟模块化也是无缝对接的。

我尝试过拓展现有的模块打包工具，但是无法实现所有目标。

### 目标

 - 将依赖树分割成可以命令加载的块
 - 保持初始加载时间短
 - 每个静态资源应该能够成为模块
 - 能够将第三方库结合成模块的能力
 - 自定义模块加载器几乎各个部分的能力
 - 适合大项目

### webpack有什么不同

[代码分割](http://webpack.github.io/docs/code-splitting.html)

webpack有两种依赖类型：同步和异步。异步的依赖充当分割的点并形成新的块。当块的树被优化，一个文件就发射每一个块。

[loaders](http://webpack.github.io/docs/loaders.html)

webpack默认处理javascript，但是loaders用来把资源转变为javascript，这样做每个资源都能构成模块。

**强大的解析能力**

webpack有一个非常强大的解析器用来处理几乎所有第三方的类库。它甚至允许在依赖引入时使用表达式`require("./templates/" + name + ".jade")`。webpack能处理普遍的模块风格： CommonJs和AMD

[插件系统](http://webpack.github.io/docs/plugins.html)

webpack产出了丰富的插件系统。多数内部特征都在插件系统基础上开发，这允许你根据自己需求自定义webpack，同时也支持贡献通用的开源插件。
