# 代码规范与代码检查工具

遵循良好的代码规范是很有必要的。随着业务的发展，项目的持续集成，代码的复杂性会变得越来越高。在业务逻辑变得越来越复杂的时候，如果没有统一的规范进行约束，代码的可阅读性会越来越差，这样拓展和维护都将会变得非常棘手。遵守代码规范，能够让多人开发维护的项目代码风格保持统一，提高可阅读性，同时也能够在编码过程中规避掉一些低级错误和需要避开的坑，降低出错概率。

## 百度前端编码规范

百度提供了一套包括 `Javascript`、`HTML`、`CSS` 等一系列前端编码规范，是全公司前端所遵循的一套标准，并且在不断地更新和完善。

百度前端代码规范相应的链接地址：

- [百度前端编码规范](https://github.com/ecomfe/spec)
    - [Javascript编码规范](https://github.com/ecomfe/spec/blob/master/javascript-style-guide.md)
    - [Javascript编码规范 - ESNext补充篇](https://github.com/ecomfe/spec/blob/master/es-next-style-guide.md)
    - [HTML编码规范](https://github.com/ecomfe/spec/blob/master/html-style-guide.md)
    - [CSS编码规范](https://github.com/ecomfe/spec/blob/master/css-style-guide.md)
    - [Less编码规范](https://github.com/ecomfe/spec/blob/master/less-code-style.md)
    - [E-JSON数据传输标准](https://github.com/ecomfe/spec/blob/master/e-json.md)
    - [模块和加载器规范](https://github.com/ecomfe/spec/blob/master/module.md)
    - [包结构规范](https://github.com/ecomfe/spec/blob/master/package.md)
    - [项目目录结构规范](https://github.com/ecomfe/spec/blob/master/directory.md)
    - [图表库标准](https://github.com/ecomfe/spec/blob/master/chart.md)
    - [react编码规范](https://github.com/ecomfe/spec/blob/master/react-style-guide.md)

## 代码检查工具 FECS

**FECS** 是基于百度前端编码规范的代码检查工具，具有“灵活”、“高效”、“齐全”的特点，包含 HTML、CSS、JavaScript 与 Less 代码的检查与修复等功能。

FECS 的官网是：[fecs.baidu.com](http://fecs.baidu.com)

FECS 详细的使用方法请参考：[快速开始](http://fecs.baidu.com/api)

### FECS 安装

FECS 的安装过程也相当简单，通过 npm 进行如下安装即可：

```bash
[sudo] npm install fecs -g
```

### 代码检查

在使用的时候只需要在待检查的项目根目录的命令行输入 `fecs` 即可实现代码检查：

```bash
fecs
fecs path
fecs path/to/file
fecs check --help
```

如果代码完全符合规范，将提示如下信息：

![fecs success](./images/spec-fecs-success.jpg)

如果代码存在不符合规范的地方，会将有问题的代码文件和对应的行号信息打印出来：

![fecs fail](./images/spec-fecs-faile.jpg)

### 代码修复

代码自动修复可以通过 `fecs format` 命令实现：

```bash
fecs format src --output=fixed
fecs format src --replace
fecs format --help
```

### 相关插件

FECS 提供了一系列编辑器和编译工具插件帮助开发者提高开发效率：

- [VIM](https://github.com/hushicai/fecs.vim)
- [WebStorm](https://github.com/leeight/Baidu-FE-Code-Style#webstorm)
- [Eclipse](https://github.com/ecomfe/fecs-eclipse)
- Sublime Text 2/3
    - [Baidu FE Code Style](https://github.com/leeight/Baidu-FE-Code-Style)
    - [Sublime Helper](https://github.com/baidu-lbs-opn-fe/Sublime-fecsHelper)
    - [SublimeLinter-contrib-fecs](https://github.com/robbenmu/SublimeLinter-contrib-fecs)
- [Visual Studio Code](https://github.com/21paradox/fecs-visual-studio-code)
- [Atom](https://github.com/8427003/atom-fecs)
- [Grunt](https://github.com/ecomfe/fecs-grunt)
- [Gulp](https://github.com/ecomfe/fecs-gulp)
- [Webpack](https://github.com/ecomfe/fecs-loader)
- [Git Hook](https://github.com/cxtom/fecs-git-hooks)
- [Kudo](https://github.com/ecomfe/kudo)

## 相关链接

- [百度前端编码规范](https://github.com/ecomfe/spec)
- [FECS 官网](http://fecs.baidu.com)
- [前端代码风格检查套件 FECS](http://efe.baidu.com/blog/fecs)