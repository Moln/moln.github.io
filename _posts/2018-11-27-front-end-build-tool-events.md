---
layout: page
title: 前端打包工具一些事件
---

前端打包工具一些事件
==================

本文描述前端构建工具组件废弃维护事件, 希望能帮助读者选择正确的组件

Webpack
-------

Webpack v4说起

- 执行 `webpack` 命令, 需要依赖 `webpack-cli` 或者 `webpack-command`
- `webpack` 构建 server 也有两者选择, `webpack-serve` 和 `webpack-dev-server`

### webpack-command & webpack-cli

`webpack-command` 起初是与 `webpack-cli` 竞争, 定位为更简洁的cli工具

- 2018-09-11 `webpack-command` 官方不在维护, 建议更改为 `webpack-cli`, 很难同时维护 `webpack-cli` 或者 `webpack-command`, 
更多详细见 [webpack-command github](https://github.com/webpack-contrib/webpack-command) DEPRECATED 部分

### webpack-serve & webpack-dev-server

`webpack-serve` 起初是与 `webpack-dev-server` 竞争, 定位为更简洁的dev server工具

`webpack-dev-server` 2018-08-22 之前, 官方处于仅修复bug不添加新功能状态

- 2018-08-22 之后`webpack-serve`不在维护, 与 `webpack-command` 同样原因, 建议改用 `webpack-dev-server`. 
  `webpack-dev-server` 也重新开启了维护添加新功能特性状态.

[webpack-serve 废弃原因见 DEPRECATED 部分](https://github.com/webpack-contrib/webpack-serve) 

### webpack v4.26.1 2018-09-10

压缩插件原本使用 `uglifyjs-webpack-plugin` 改用 `terser-webpack-plugin` 

https://github.com/webpack/webpack/commit/311a7285d36b38bada46102967c431e93ff48a89#diff-7473dd12403a310ad8cf18d0ab9f10b9

uglify-es & terser
------------------

- `uglify-es` 用于压缩 ES6+ 代码, 2018-05 声明不在维护
- `uglify-js` 压缩不支持 ES6+
- `terser` Fork `uglify-es` 压缩支持 ES6+, 兼容 uglify-js 配置选项.

