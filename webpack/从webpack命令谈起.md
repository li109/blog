# webpack4.0源码学习（一）：从webpack命令谈起
&emsp;&emsp;在开发基于 webpack 构建的项目时，一般都使用 npm run script 命令行的方式来完成启动本地服务器、打包等功能。探究从输入命令到最终完成构建的过程，是学习 webpack4.0 源码不错的突破口。<br/>
&emsp;&emsp;本系列文章基于 webpack 4.30.0 版本以及 webpack-cli 3.3.1版本。<br/>
## 一、npm run script
&emsp;&emsp;官网上的入门教程提供一个简单的例子，首先执行 npm init -y 完成项目初始化，然后通过 npm install webpack webpack-cli --save-dev 命令安装 webpack 以及 webpack-cli，最后修改 package.json 中的 scripts 属性如下：<br/>
```json
"scripts": {
    "build": "webpack"
}
```
&emsp;&emsp;执行 npm run build 命令，会根据平台的不同调用 /node_modules/.bin/ 下不同的对应脚本文件，在类Unix系统下调用 /node_modules/.bin/webpack shell 脚本，在Windows 系统下调用 /node_modules/.bin/webpack.cmd dos文件。<br/>
&emsp;&emsp;/node_modules/.bin/webpack 源码如下：<br/>
```js
#!/bin/sh
basedir=$(dirname "$(echo "$0" | sed -e 's,\\,/,g')")

case `uname` in
    *CYGWIN*) basedir=`cygpath -w "$basedir"`;;
esac

if [ -x "$basedir/node" ]; then
  "$basedir/node"  "$basedir/../_webpack@4.30.0@webpack/bin/webpack.js" "$@"
  ret=$?
else 
  node  "$basedir/../_webpack@4.30.0@webpack/bin/webpack.js" "$@"
  ret=$?
fi
exit $ret
```
&emsp;&emsp;/node_modules/.bin/webpack.cmd 源码如下：<br/>
```js
@IF EXIST "%~dp0\node.exe" (
  "%~dp0\node.exe"  "%~dp0\..\_webpack@4.30.0@webpack\bin\webpack.js" %*
) ELSE (
  @SETLOCAL
  @SET PATHEXT=%PATHEXT:;.JS;=;%
  node  "%~dp0\..\_webpack@4.30.0@webpack\bin\webpack.js" %*
)
```
&emsp;&emsp;这两个文件的功能是一样的：使用 node 命令执行 /node_modules 中安装的 webpack 文件夹中的 /bin/webpack.js。<br/>
## 二、/webpack/bin/webpack.js
&emsp;&emsp;/webpack/bin/webpack.js 的功能是检测是否安装 webpack-cli 与 webpack-command 包，然后根据具体情况分别进行处理。<br/>
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
&emsp;&emsp;最常见的是只安装 webpack-cli 包，如果是这样则加载执行 /webpack-cli/bin/cli.js 文件。<br/>
### 3、webpack-cli 与 webpack-command 都已安装
&emsp;&emsp;如果两个包都已安装，则会显示如下信息，提醒用户只需一个 CLI 包，可以删除其中一个包或者直接通过二进制文件使用它们。<br/>
> You have installed webpack-cli and webpack-command together. To work with the "webpack" command you need only one CLI package, please remove one of them or use them directly via their binary.<br/>

## 三、/webpack-cli/bin/cli.js
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
## 四、/webpack/lib/webpack.js
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
