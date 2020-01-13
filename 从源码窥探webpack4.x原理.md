# 「搞点硬货」从源码窥探Webpack4.x原理


随着多终端设备的迅速普及，Web前端开发的复杂性和应用场景日益扩大，Webpack在前端构建演变的工程化浪潮中担当起了针对不同应用场景打包的大任。
如今，Webpack可谓是JavaScript社区最伟大的项目之一。


本文力争从源码层面窥探Webpack的实现原理。文中出示了核心的代码块并注释了相应的path，如果你也想揭开Webpack神秘的面纱，那就打开一份源码跟随本文一起享受一次秃头的快乐。


## Webpack本质
Webpack的本质是什么呢？可能有的同学已经知道了，

`Webpack本质上一种基于事件流的编程范例，其实就是一系列的插件运行。`

Webpack主要使用`Compiler`和`Compilation`两个类来控制Webpack的整个生命周期。他们都继承了Tapabel并且通过Tapabel来注册了生命周期中的每一个流程需要触发的事件。

## Tapabel
[Tapabel](https://github.com/webpack/tapable)是一个类似于 Node.js 的 EventEmitter 的库，主要是控制钩子函数的发布与订阅，是Webpack插件系统的大管家。

### Tapabel提供的钩子及示例
Tapable库为插件提供了很多 Hook以便挂载。

```javascript
const {
    SyncHook,                   // 同步钩子
    SyncBailHook,               // 同步熔断钩子
    SyncWaterfallHook,          // 同步流水钩子
    SyncLoopHook,               // 同步循环钩子
    AsyncParalleHook,           // 异步并发钩子
    AsyncParallelBailHook,      // 异步并发熔断钩子
    AsyncSeriesHook,            // 异步串行钩子
    AsyncSeriesBailHook,        // 异步串行熔断钩子
    AsyncSeriesWaterfallHook     // 异步串行流水钩子
} = require("tapable");
```
Tabpack 提供了`同步&异步`绑定钩子的方法，方法如下所示：
|Async|Sync|
|:---|:---|
|绑定：tapAsync/tapPromise/tap|绑定：tap||
|执行：callAsync/promise|执行：call||

### Tabpack简单示例
```javascript
const demohook = new SyncHook(["arg1", "arg2", "arg3"]);
// 绑定事件到webpack事件流
demohook.tap("hook1",(arg1, arg2, arg3) => console.log(arg1, arg2, arg3)) // 1 2 3
// 执行绑定的事件
demohook.call(1,2,3)
```

## 源码解读
### 初始化启动之Webpack的入口文件

追本溯源，第一步我们要找到Webpack的入口文件。

当通过命令行启动Webpack后，npm会让命令行工具进入node_modules\.bin 目录。

然后查找是否存在 webpack.sh 或者 webpack.cmd 文件，如果存在，就执行它们，不存在就会抛出错误。

实际的入口文件是：`node_modules/webpack/bin/webpack.js`，让我们来看一下里面的核心函数。

```javascript
// node_modules/webpack/bin/webpack.js
// 正常执行返回
process.exitCode = 0;    
// 运行某个命令                               
const runCommand = (command, args) => {...}
// 判断某个包是否安装
const isInstalled = packageName => {...}
// webpack可用的CLI：webpacl-cli和webpack-command
const CLIs = {...}
// 判断是否两个CLI是否安装了
const installedClis = CLIs.filter(cli=>cli.installed);
// 根据安装数量进行处理
if (installedClis.length === 0) {...} else if 
 (installedClis.length === 1) {...} else {...}
```
启动后，Webpack最终会找到 `webpack-cli /webpack-command `的 npm 包，并且 执行 CLI。


### webpack-cli
搞清楚了Webpack启动的入口文件后，接下来让我们把目光转移到webpack-cli，看看它做了哪些事儿。

- 引入 yargs，对命令行进行定制
- 分析命令行参数，对各个参数进行转换，组成编译配置项
- 引用webpack，根据配置项进行编译和构建

`webpack-cli` 会处理不需要经过编译的命令。

```javascript
// node_modules/webpack-cli/bin/cli.js
const {NON_COMPILATION_ARGS} = require("./utils/constants");
const NON_COMPILATION_CMD = process.argv.find(arg => {
    if (arg === "serve") {
        global.process.argv = global.process.argv.filter(a => a !== "serve");
        process.argv = global.process.argv;
    }
    return NON_COMPILATION_ARGS.find(a => a === arg);
});
if (NON_COMPILATION_CMD) {
    return require("./utils/prompt-command")(NON_COMPILATION_CMD,...process.argv);
}
```

`webpack-cli`提供的不需要编译的命令如下。

```javascript
// node_modules/webpack-cli/bin/untils/constants.js
const NON_COMPILATION_ARGS = [
    "init",                 // 创建一份webpack配置文件
    "migrate",              // 进行webpack版本迁移
    "add",                  // 往webpack配置文件中增加属性
    "remove",               // 往webpack配置文件中删除属性
    "serve",                // 运行webpack-serve
    "generate-loader",      // 生成webpack loader代码
    "generate-plugin",      // 生成webpack plugin代码
    "info"                  // 返回与本地环境相关的一些信息
];
```
webpack-cli 使用命令行工具包yargs。

```javascript
// node_modules/webpack-cli/bin/config/config-yargs.js
const {
    CONFIG_GROUP,
    BASIC_GROUP,
    MODULE_GROUP,
    OUTPUT_GROUP,
    ADVANCED_GROUP,
    RESOLVE_GROUP,
    OPTIMIZE_GROUP,
    DISPLAY_GROUP
} = GROUPS;
```
webpack-cli对配置文件和命令行参数进行转换最终生成配置选项参数 options，最终会根据配置参数实例化webpack对象，然后执行构建流程。

除此之外，让我们回到`node_modules/webpack/lib/webpack.js`里来看一下Webpack还做了哪些准备工作。
```javascript
// node_modules/webpack/lib/webpack.js
const webpack = (options, callback) => {
    ...
    options = new WebpackOptionsDefaulter().process(options);
    compiler = new Compiler(options.context);
    new NodeEnvironmentPlugin().apply(compiler);
    ...
    compiler.options = new WebpackOptionsApply().process(options, compiler);
    ...
    webpack.WebpackOptionsDefaulter = WebpackOptionsDefaulter;
    webpack.WebpackOptionsApply = WebpackOptionsApply;
    ...
    webpack.NodeEnvironmentPlugin = NodeEnvironmentPlugin;
}
```
WebpackOptionsDefaulter的功能是设置一些默认的Options(代码比较多不贴了，大家自行查看`node_modules/webpack/lib/WebpackOptionsDefaulter.js`)。



```javascript
// node_modules/webpack/lib/node/NodeEnvironmentPlugin.js
class NodeEnvironmentPlugin {
  apply(compiler) {
      ...		
      compiler.hooks.beforeRun.tap("NodeEnvironmentPlugin", compiler => {
	  if (compiler.inputFileSystem === inputFileSystem) inputFileSystem.purge();
      });
  }
}
```
从上面的代码我们可以知道，`NodeEnvironmentPlugin`插件监听了beforeRun钩子，它的作用是清除缓存。

### WebpackOptionsApply
`WebpackOptionsApply`会将所有的配置options参数转换成webpack内部插件。

使用默认插件列表
- output.library -> LibraryTemplatePlugin
- externals -> ExternalsPlugin
- devtool -> EvalDevtoolModulePlugin, SourceMapDevToolPlugin
- AMDPlugin, CommonJsPlugin
- RemoveEmptyChunksPlugin

```javascript
// node_modules/webpack/lib/WebpackOptionsApply.js
new EntryOptionPlugin().apply(compiler);
compiler.hooks.entryOption.call(options.context, options.entry);
```
实际上，插件最后都会变成`compiler`对象上的实例。

### EntryOptionPlugin
接下来让我们进入`EntryOptionPlugin`插件，看看它做了哪些事儿。
```javascript
// node_modules/webpack/lib/EntryOptionPlugin.js
module.exports = class EntryOptionPlugin {
    apply(compiler) {
        compiler.hooks.entryOption.tap("EntryOptionPlugin", (context, entry) => {
	    if (typeof entry === "string" || Array.isArray(entry)) {
	        itemToPlugin(context, entry, "main").apply(compiler);
	    } else if (typeof entry === "object") {
	        for (const name of Object.keys(entry)) {
		    itemToPlugin(context, entry[name], name).apply(compiler);
	        }
	    } else if (typeof entry === "function") {
	        new DynamicEntryPlugin(context, entry).apply(compiler);
	    }
	    return true;
	});
    }
};
```
如果是数组，则转换成多个entry来处理，如果是对象则转换成一个个entry来处理。

如上述代码所示。

compiler实例化是在`node_modules/webpack/lib/webpack.js`里完成的。通过`EntryOptionPlugin`插件进行参数校验。通过`WebpackOptionsDefaulter`将传入的参数和默认参数进行合并成为新的options，创建compiler，以及相关plugin，最后通过
`WebpackOptionsApply`将所有的配置options参数转换成Webpack内部插件。

不要急，还没完事。

再次来到我们的`node_modules/webpack/lib/webpack.js`中
```javascript
if (options.watch === true || (Array.isArray(options) && options.some(o => o.watch))) {
  const watchOptions = Array.isArray(options)
	? options.map(o => o.watchOptions || {})
	: options.watchOptions || {};
	return compiler.watch(watchOptions, callback);
}
compiler.run(callback);
```

实例compiler后会根据options的watch判断是否启动了`watch`，如果启动watch了就调用`compiler.watch`来监控构建文件，否则启动`compiler.run`来构建文件。

## 编译构建
### compile
首先会实例化`NormalModuleFactory`和`ContextModuleFactory`。然后进入到`run`方法。

```javascript
// node_modules/webpack/lib/Compiler.js
run(callback) { 
    ...
    // beforeRun 如上文NodeEnvironmentPlugin插件清除缓存
    this.hooks.beforeRun.callAsync(this, err => {
        if (err) return finalCallback(err);
        // 执行run Hook开始编译
        this.hooks.run.callAsync(this, err => {
            if (err) return finalCallback(err);
            this.readRecords(err => {
                if (err) return finalCallback(err);
                // 执行compile
                this.compile(onCompiled);
            });
        });
    });
}
```
在执行`this.hooks.compile`之前会执行`this.hooks.beforeCompile`，来对编译之前需要处理的插件进行执行。紧接着`this.hooks.compile`执行后会实例化`Compilation`对象。

```javascript
// node_modules/webpack/lib/compiler.js
compile(callback) {
    const params = this.newCompilationParams();
    this.hooks.beforeCompile.callAsync(params, err => {
	if (err) return callback(err);
	// 进入compile阶段
	this.hooks.compile.call(params);
	const compilation = this.newCompilation(params);
	// 进入make阶段
	this.hooks.make.callAsync(compilation, err => {
	    if (err) return callback(err);
	    compilation.finish(err => {
		if (err) return callback(err);
		// 进入seal阶段
		compilation.seal(err => {
		    if (err) return callback(err);
		    this.hooks.afterCompile.callAsync(compilation, err => {
			if (err) return callback(err);
			return callback(null, compilation);
		    })
 		})
	    })
	})
    })
}
```

## make      
一个新的`Compilation`创建完毕，将从Entry开始读取文件，根据文件类型和配置的Loader对文件进行编译，编译完成后再找出该文件依赖的文件，递归的编译和解析。

我们来看一下make钩子被监听的地方。

如代码中注释所示，`addEntry是make构建阶段真正开始的标志`。
```javascript
// node_modules/webpack/lib/SingleEntryPlugin.js
compiler.hooks.make.tapAsync(
    "SingleEntryPlugin",
    (compilation, callback) => {
	const { entry, name, context } = this;
	cosnt dep = SingleEntryPlugin.createDependency(entry, name);
	// make构建阶段开始标志 
	compilation.addEntry(context, dep, name, callback);
    }
)
```
addEntry实际上调用了`_addModuleChain`方法，_addModuleChain方法将模块添加到依赖列表中去，同时进行模块构建。构建时会执行如下函数。

```javascript
// node_modules/webpack/lib/Compilation.js
// addEntry -> addModuleChain
_addModuleChain(context, dependency, onModule, callback) {
...
this.buildModule(module, false, null, null, err => {
	...
})
...
}
```

如果模块构建完成，会触发`finishModules`。

```javascript
// node_modules/webpack/lib/Compilation.js
finish(callback) {
    const modules = this.modules;
    this.hooks.finishModules.callAsync(modules, err => {
        if (err) return callback(err);
	for (let index = 0; index < modules.length; index++) {
	    const module = modules[index];			
            this.reportDependencyErrorsAndWarnings(module, [module]);
        }
        callback();
    })
}
```
### Module
Module包括`NormalModule(普通模块)`、`ContextModule(./src/a  ./src/b)`、`ExternalModule(module.exports=jQuery)`、`DelegatedModule(manifest)`以及`MultiModule(entry:['a', 'b'])`。

本文以NormalModule(普通模块)为例子，看一下构建(`Compilation`)的过程。
- 使用 loader-runner 运行 loaders 
- Loader转换完后，使用 acorn 解析生成AST
- 使用 ParserPlugins 添加依赖

### loader-runner
```javascript
// node_modules/webpack/lib/NormalModule.js
const { getContext, runLoaders } = require("loader-runner");
doBuild(){
    ...
    runLoaders(
        ...
    )
    ...
}
...
try {
    const result = this.parser.parse()
}
```
`doBuild`会去加载资源，doBuild中会传入资源路径和插件资源去调用`loader-runner`插件的runLoaders方法去加载和执行loader。

### acorn
```javascript
// node_modules/webpack/lib/Parser.js
const acorn = require("acorn");
```
使用`acorn`解析转换后的内容，输出对应的抽象语法树(AST)。

```javascript
// node_modules/webpack/lib/Compilation.js
this.hooks.buildModule.call(module);
...
if (error) {
    this.hooks.failedModule.call(module, error);
    return callback(error);
}
this.hooks.succeedModule.call(module);
return callback();
```

成功就触发`succeedModule`，失败就触发`failedModule`。

最终将上述阶段生成的产物存放到`Compilation.js的this.modules = [];`上。

完成后就到了seal阶段。

这里补充介绍一下Chunk生成的算法。

### Chunk生成算法
1.webpack首先会将entry中对应的module都生成一个新的chunk。

2.遍历module的依赖列表，将依赖的module也加入到chunk中。

3.如果一个依赖module是动态引入的模块，会根据这个module创建一个新的chunk，继续遍历依赖。

4.重复上面的过程，直至得到所有的chunk。



## seal
所有模块及其依赖的模块都通过Loader转换完成，根据依赖关系开始生成Chunk。

`seal`阶段也做了大量的的优化工作，进行了hash的创建以及对内容进行生成(`createModuleAssets`)。
```javascript
// node_modules/webpack/lib/Compilation.js
this.createHash();
this.modifyHash();
this.createModuleAssets();
```
```javascript
// node_modules/webpack/lib/Compilation.js
createModuleAssets(){
    for (let i = 0; i < this.modules.length; i++) {
	const module = this.modules[i];
	if (module.buildInfo.assets) {
	    for (const assetName of Object.keys(module.buildInfo.assets)) {
		const fileName = this.getPath(assetName);
		this.assets[fileName] = module.buildInfo.assets[assetName];
		this.hooks.moduleAsset.call(module, fileName);
	    }
	}
    }
}
```
seal阶段经历了很多的优化，比如`tree shaking`就是在这个阶段执行。最终生成的代码会存放在`Compilation`的`assets`属性上。

## emit
将输出的内容输出到磁盘，创建目录生成文件，文件生成阶段结束。

```javascript
// node_modules/webpack/lib/compiler.js
this.hooks.emit.callAsync(compilation, err => {
    if (err) return callback(err);
    outputPath = compilation.getPath(this.outputPath);
    this.outputFileSystem.mkdirp(outputPath, emitFiles);
})
```

## 实现一个简易的Webpack
为了能够更深入的理解Webpack的整体流程，我们可以动手来实现一个简易的Webpack。

[简易的Webpack源码地址](https://github.com/Geekhyt/simplepack) 仓库地址，欢迎Star～

## 总结

Webpack在启动阶段对配置参数和命令行参数以及默认参数进行了合并，并进行了插件的初始化工作。完成初始化的工作后调用Compiler的`run`开启Webpack编译构建过程，构建主要流程包括`compile`、`make`、`build`、`seal`、`emit`等阶段。

当然，Webpack源码还包括很多具体的实现细节，通过一篇文章是总结不完的，大家感兴趣的可以进一步学习。


## 参考
- 深入浅出Webpack
- 玩转Webpack
- [webpack4源码分析](https://juejin.im/post/5c1859745188254fef232ead#heading-19)








