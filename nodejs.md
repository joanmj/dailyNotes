# 安装

bash on windows 上安装版本10的node:

```bash
wget -qO- https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs
```

每次安装后之前安装的gitbook会丢失, 需要重新安装一次:

```bash
npm install gitbook-cli -g
```

## 开发环境

Atom

1. 安装扩展:

   1. Script: 运行js

      |      | Mac    | Windows      |
      | ---- | ------ | ------------ |
      | 运行 | cmd+i  | ctrl+shift+b |
      | 结束 | ctrl+c | ctrl+q       |

   2. atom-ternjs: 补全js

      1. Package > Atom Ternjs > Configure project配置;

      2. 配置常用字段:

         | 字段名      | 含义                                              |
         | ----------- | ------------------------------------------------- |
         | ecmaVersion | 选择 ECMAScript 版本                              |
         | libs        | browser 表示原生 js 补全，jquery 代表 jQuery 补全 |
         | loadEagerly | 指定加载解析的 js 文件                            |
         | dontLoad    | 排除加载的文件                                    |
         | plugins     | ternjs 使用的插件，配置的扩展补全的库等           |

      3. Save & Restart server 之后会在项目根目录生成 .tern-project 文件,此时输入已经有了自动补全效果了;

   3. atom-beautify:自动格式化美化代码; 快捷键Ctrl+Alt+B




# NPM

## 介绍

NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：

* 允许用户从NPM服务器**下载**别人编写的**第三方包**到本地使用。
* 允许用户从NPM服务器**下载并安装别人编写的命令行程序**到本地使用。
* 允许用户将自己编写的包或命令行程序**上传到NPM服务器**供别人使用。

npm 安装 Node.js 模块语法格式如下： 

```BASH
$ npm install <Module Name>
```

安装好之后，express 包就放在了工程目录下的 node_modules 目录中，因此在代码中只需要通过 require('express') 的方式就好，无需指定第三方包路径。 

```JS
var express = require('express');
```

## 安装命令:

```bash
npm install express         #本地安装:
```

本地安装将安装包放在 ./node_modules 下
可以通过 require() 来引入本地安装的包。



```bash
npm install express -g   # 全局安装
```

全局安装将安装包放在 /usr/local 下或者你 node 的安装目录。可以直接在命令行里使用。

希望具备两者功能，则需要在两个地方安装它或使用 `npm link`。

## 查看安装包:

命令:

```bash
#使用以下命令查看：
 npm ls
```

查看所有全局安装的模块：

```bash
npm list -g
```

查看某个模块的版本号，可以使用命令如下：

```
npm list grunt
```



## 卸载:

```bash
npm uninstall express
```

查找包:

```bash
$ npm search express
```
创建包, package.json 文件是必不可少的。使用 NPM 生成 package.json 文件，生成的文件包含了基本的结果
```bash
$ npm init
```

## 在 npm 资源库中注册用户：
用以下命令（使用邮箱注册）
```bash
$ npm adduser
Username: mcmohd
Password:
Email: (this IS public) mcmohd@gmail.com
```
用以下命令来发布模块：
```bash
$ npm publish
```

发布的版本号含义:
语义版本号分为X.Y.Z三位，分别代表主版本号、次版本号和补丁版本号。当代码变更时，版本号按以下原则更新。
如果只是修复bug，需要更新Z位。
如果是新增了功能，但是向下兼容，需要更新Y位。
如果有大变动，向下不兼容，需要更新X位。
在申明第三方包依赖时，除了可依赖于一个固定版本号外，还可依赖于某个范围的版本号。例如"argv": "0.0.x"表示依赖于0.0.x系列的最新版argv。

## NPM命令

查看某条命令的详细帮助
```bash
npm help <command> #例如npm help install。
```
先在本地安装当前命令行程序，可用于发布前的本地测试:
在package.json所在目录下使用:
先在本地安装当前命令行程序，可用于发布前的本地测试。
```bash
npm install . -g
```
把当前目录下node_modules子目录里边的对应模块更新至最新版本:
```bash
npm update <package>可以
```
把全局安装的对应命令行程序更新至最新版:
```bash
npm update <package> -g
```
清空NPM本地缓存，用于对付使用相同版本号发布新版本代码的人:
```bash
npm cache clear
```
撤销发布自己发布过的某个版本代码:
```bash
npm unpublish <package>@<version>
```
## 使用淘宝 NPM 镜像:
```bash
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```
这样就可以使用 cnpm 命令来安装模块了：
```bash
$ cnpm install [name]
```

