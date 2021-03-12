# webpack5.0源码解析（一）：webpack-cli
&emsp;&emsp;从`webpack4.0`开始，使用`webpack`的同时需要安装`webpack-cli`。在对`webpack5.0`进行深度解析前，我们先来探究`webpack-cli`的功能以及原理。<br/>
&emsp;&emsp;本系列文章基于`webpack 5.24.2`版本以及`webpack-cli 4.5.0`版本。<br/>
## 一、从webpack命令谈起
&emsp;&emsp;官网上的入门教程提供一个简单的例子，首先执行`npm init -y`完成项目初始化，然后通过`npm install webpack webpack-cli --save-dev`命令安装`webpack`以及`webpack-cli`，最后修改`package.json`中的`scripts`属性如下：<br/>
```json
"scripts": {
    "build": "webpack"
}
```
&emsp;&emsp;执行`npm run build`命令就会完成简单的打包功能，那么这条命令具体执行的过程是什么呢？我们来详细的看一下。
&emsp;&emsp;在执行`npm run`命令时会自动新建一个`Shell`，在这个`Shell`里面执行指定的脚本命令。新建的这个`Shell`，会将当前目录的`node_modules/.bin`子目录加入`PATH`变量，执行结束后，再将`PATH`变量恢复原样。也就是说，当前目录的`node_modules/.bin`子目录里面的所有脚本，都可以直接用脚本名调用，而不必加上路径。<br/>
&emsp;&emsp;在上文示例中执行`npm run build`命令就是执行`node_modules/.bin`下的`webpack`脚本。根据平台的不同调用不同的对应脚本文件，在类Unix系统下调用`/node_modules/.bin/webpack`shell脚本，在Windows系统下调用`/node_modules/.bin/webpack.cmd`dos文件。两个文件的功能是一样的，下面我们选择`Shell`文件来进行分析。<br/>
```shell
#!/bin/sh
basedir=$(dirname "$(echo "$0" | sed -e 's,\\,/,g')")

case `uname` in
    *CYGWIN*|*MINGW*|*MSYS*) basedir=`cygpath -w "$basedir"`;;
esac

if [ -x "$basedir/node" ]; then
  "$basedir/node"  "$basedir/../webpack/bin/webpack.js" "$@"
  ret=$?
else 
  node  "$basedir/../webpack/bin/webpack.js" "$@"
  ret=$?
fi
exit $ret
```
&emsp;&emsp;这段`shell`的主要功能是使用`node`命令执行`/node_modules`中安装的`webpack.js`文件。在当前情况下就是在`node`环境下执行`/node_modules/webpack/bin/webpack.js`文件。<br/>
&emsp;&emsp;由此看来，`/bin/webpack.js`就是使用`webpack`打包的入口文件，其主干代码如下：<br/>
```js
const runCommand = (command,args) => {/*...*/}
const isInstalled = packageName => {/*...*/}

const runCli = cli => {
	const path = require("path")
	const pkgPath=require.resolve(`${cli.package}/package.json`)
	const pkg = require(pkgPath)
	require(path.resolve(path.dirname(pkgPath), pkg.bin[cli.binName]))
}

const cli = {/*...*/}
if (!cli.installed) {
  /*...*/
} else {
	runCli(cli)
}
```
&emsp;&emsp;我们的主要目的是搞清楚输入命令后的流程，暂时抛开具体细节后面再做讨论。该文件的主要功能就是判断`webpack-cli`是否安装，如果安装则执行`/node_modules/webpack-cli/bin/cli.js`文件。<br/>
&emsp;&emsp;`cli.js`的核心代码如下所示：<br/>
```js
/*...*/
const runCLI = require('../lib/bootstrap')

if (utils.packageExists('webpack')) {
  runCLI(process.argv, originalModuleCompile)
}
```
&emsp;&emsp;`cli.js`的主要功能是检测`webpack`是否下载，如果下载则执行`runCLI`函数，该函数实现在`/lib/bootstrap.js`文件中。<br/>
```js
const WebpackCLI = require('./webpack-cli')
const runCLI = async (args,originalModuleCompile) => {
  /*...*/
  const cli = new WebpackCLI()
  await cli.run(args)
}
```
&emsp;&emsp;到此为止，整个流程就比较清晰了：执行`npm script`，实质就是执行`WebpackCLI`的实例对象的`run`方法。<br/>
&emsp;&emsp;函数`WebpackCLI`的实现在`/lib/webpack-cli.js`中。<br/>
```js
class WebpackCLI {
  constructor() {
    this.webpack = require('webpack')
    this.program = program
    /*...*/
  }
}
```
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
## 二、webpack 包的 /bin/webpack.js
&emsp;&emsp;webpack包的 /bin/webpack.js 的功能是检测是否安装 webpack-cli 与 webpack-command 包，然后根据具体情况分别进行处理。<br/>
&emsp;&emsp;webpack4.0 强制要求必须使用一种 CLI 包，官网上甚至直接说必须安装 webpack-cli，因为 webpack-command 现在很不推荐使用，源码中保留这种判断是为了防止依然有用户使用 webpack-command 的情况。<br/>
&emsp;&emsp;webpack-command 被弃用的原因主要有以下五点：<br/>
> 1、webpack-cli 非常稳定，具有更多功能。<br/>
> 2、两个CLI对开发人员具有误导性。<br/>
> 3、很难保持两个具有相同目的的包装。<br/>
> 4、webpack-command 包已经停止更新。<br/>
> 5、webpack-command 的大多数功能已经在 webpack-cli 中实现。<br/>

### 1、webpack-cli 与 webpack-command 都没安装
&emsp;&emsp;如果两种包都没有安装，则首先提示如下信息：<br/>
> One CLI for webpack must be installed. These are recommended choices, delivered as separate packages:<br/>
> \- webpack-cli (https://github.com/webpack/webpack-cli)<br/>
>   The original webpack full-featured CLI.<br/>

&emsp;&emsp;然后检测是否使用 yarn 包管理工具，如果是，则提示用户是否使用 yarn 安装 webpack-cli 包：<br/>
> We will use "yarn" to install the CLI via "yarn add -D".<br/>
> Do you want to install 'webpack-cli' (yes/no):<br/>

&emsp;&emsp;如果使用 npm 包管理工具，则提示用户是否使用 npm 安装 webpack-cli 包：<br/>
> We will use "npm" to install the CLI via "npm install -D".<br/>
> Do you want to install 'webpack-cli' (yes/no):<br/>

### 2、webpack-cli 与 webpack-command 安装其一
&emsp;&emsp;如果安装其中一个包，首先获取该包 package.json 文件的绝对路径，然后加载 package.json 文件，再根据 package.json 文件中的 bin 选项，加载该包的执行文件。<br/>
&emsp;&emsp;最常见的是只安装 webpack-cli 包，如果是这样则加载执行 webpack-cli 包的 /bin/cli.js 文件。本例中是加载执行 /node_modules/_webpack-cli@3.3.1@webpack-cli/bin/cli.js 文件<br/>
### 3、webpack-cli 与 webpack-command 都已安装
&emsp;&emsp;如果两个包都已安装，则会显示如下信息，提醒用户只需一个 CLI 包，可以删除其中一个包或者直接通过二进制文件使用它们。<br/>
> You have installed webpack-cli and webpack-command together. To work with the "webpack" command you need only one CLI package, please remove one of them or use them directly via their binary.<br/>

## 三、webpack-cli 包的 /bin/cli.js
&emsp;&emsp;1、引入 import-local 包，优先使用本地 webpack-cli，其次使用全局安装的包。<br/>
&emsp;&emsp;2、引入 v8-compile-cache 包，使用v8引擎的代码缓存来加快实例化时间。<br/>
&emsp;&emsp;3、添加命令行提示信息。<br/>
<!-- &emsp;&emsp;4、可以通过命令 config 指定 webpack 的配置文件；如果不输入，则默认配置文件为webpack.config.js；如果未通过 config 命令配置文件且 webpack.config.js 文件不存在，则使用默认配置。<br/> -->
&emsp;&emsp;4、可以通过命令 config 指定 webpack 的配置文件；如果不输入，则默认配置文件为webpack.config.js；如果未通过 config 命令配置文件且 webpack.config.js 不存在，则 options 对象只有 context 属性，存储项目路径。<br/>
options = require("./utils/convert-argv")(argv);<br/>
&emsp;&emsp;然后根据配置信息生成 options 选项，options 对象中包含 上下文环境、输入文件、输出文件等信息。<br/>
compiler = webpack(options);<br/>
&emsp;&emsp;5、加载 webpack ，即引入 webpack 包中的 /lib/webpack.js 文件导出函数。将 配置对象 options 传入 webpack 函数，返回值赋值给 compiler 。然后执行 compiler.run() 方法完成打包编译。<br/>
## 四、webpack 包的 /lib/webpack.js
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
