title: 通过源码学习hexo
date: 2014-12-10 12:46:12
author: 赵空暖
tags:
- hexo
categories:
- hexo
---
## 先序知识
###模块
* 参考阮一峰的[前端模块管理器简介](http://www.ruanyifeng.com/blog/2014/09/package-management.html)
* [Javascript模块化编程（一）：模块的写法](http://www.ruanyifeng.com/blog/2012/10/javascript_module.html)
* [Javascript模块化编程（二）：AMD规范](http://www.ruanyifeng.com/blog/2012/10/asynchronous_module_definition.html)
* [Javascript模块化编程（三）：require.js的用法](http://www.ruanyifeng.com/blog/2012/11/require_js.html)

###NodeJS社区
![nodejs](/image/nodejs.png)

* NodeJS 移步 → [Nodejs学习路线图](http://blog.fens.me/nodejs-roadmap/)
* npm 移步 → [快速创建基于npm的nodejs库](http://blog.fens.me/nodejs-npm-package/)
* RequireJS 移步 → [RequireJS异步模块加载](http://blog.fens.me/nodejs-requirejs/)

###hexo是什么
* 关于hexo的详细说明请移驾至[doc](http://hexo.io/docs/)
* 下载源码
```bash
  git clone git@github.com:hexojs/hexo.git
```
 看到如下目录结构
 ![hexo1](/image/hexo1.png)

* assets
	* 里面放着模板
		* scaffolds：layout模板文件目录，其中的md文件可以添加编辑
		* source：文章资源目录，该目录下的markdown和html文件均会被hexo处理。image、404文件、favicon.ico文件，CNAME文件等都应该放这里。
			* _posts：发布的文章。
		* themes：主题文件目录。
		* _config.yml：全局配置文件，详细配置请看[官方文档](http://hexo.io/docs/configuration.html)。
		* package.json：关于hexo的一些信息，如版本等。 
* lib
	* hexo的核心代码
* bin
	* 只有一个文件hexo,安装后被放在系统环境变量中,终端输入的hexo即直接调用该文件。
```bash
#!/usr/bin/env node

var args = require('minimist')(process.argv.slice(2)),
  fs = require('graceful-fs'),
  path = require('path'),
  async = require('async'),
  init = require('../lib/init'),
  cwd = process.cwd(),
  lastCwd = cwd;

// Find Hexo folder recursively
async.doUntil(
  function(next){
    var configFile = path.join(cwd, '_config.yml');

    fs.exists(configFile, function(exist){
      if (exist){
        init(cwd, args);
      } else {
        lastCwd = cwd;
        cwd = path.dirname(cwd);
        next();
      }
    });
  },
  function(){
    return cwd === lastCwd;
  },
  function(){
    init(process.cwd(), args);
  }
);
```
其中该文件调用了lib下的init.js文件`init = require('../lib/init')`进行一些初始化工作。
在init.js文件中
```javascript
var async = require('async'),
  path = require('path'),
  Hexo = require('./core'),
  Logger = require('./logger');

var defaultCallback = function(err){
  process.exit(err ? 1 : 0);
};

module.exports = function(cwd, args, callback){
  if (typeof callback !== 'function') callback = defaultCallback;

  var hexo = global.hexo = new Hexo(),
    configfile = args.config || '_config.yml';

  hexo.bootstrap(cwd, args);
  hexo.configfile = path.join(hexo.base_dir, configfile);

  async.eachSeries([
    'logger',
    'extend',
    'config',
    'update',
    'database',
    'box',
    'plugins',
    'scripts'
  ], function(name, next){
    require('./loaders/' + name)(next);
  }, function(err){
    if (err){
      if (hexo.log != null){
        return hexo.log.e(err);
      } else {
        throw err;
      }
    }
```
加载了anync ,async是一个用于流程处理的工具包，调用方法来处理有依赖关系的多个任务`async.eachSeries`的执行。详细移步[Nodejs异步流程控制Async](http://blog.fens.me/nodejs-async/)有如下任务:
1. logger :log的相关操作
2. extend
3. config :从_config.yml配置文件中加载博客配置信息
4. update :更新package.json中的包依赖
5. database :加载数据库 使用了Warehouse，这是hexo作者自己实现的一个基于JSON的数据库,详细参看[Warehouse API](http://hexo.io/api/warehouse/classes/Database.html) 实现代码点击这里[Warehouse](https://github.com/tommy351/warehouse)
6. box
7. plugins :加载插件，这里的插件对应node_modules文件夹下面的一些node模块
8. scripts :加载脚本，自动执行scripts文件夹下的JS文件，用来对hexo作扩展
这些都在lib文件夹下的对应目录下
 ![hexo2](/image/hexo2.png)

 其中查看查看lib/plugins/console下的文件：
* clean.js 清除Warehourse的缓存
* config.js 打印当前配置信息
* deploy.js 部署到线上，如在_config.yml配置了如下信息,则提交到github中
```bash
deploy:
  type: github
  repository: git@github.com:KongnuanZhao/KongnuanZhao.github.io.git
  branch: master
```
* generate.js  生成静态文件 `hexo generate`
* help.js 帮助
* index.js 这些文件都有对应的控制台命令，并通过index.js对外注册所有的命令，用于控制台的调用。
* init.js 初始化新的Hexo文件夹 `init`
* migrate.js 从其他系统迁移到hexo的搬家命令，支持RSS，Jekyll，Octopress和WordPress `hexo migrate`
* new.js  创建新文章，默认为post，可选参数为page（新页面）和 draft（草稿）`hexo new`
* publish.js 将文章发布`hexo publish`
* render.js 用于markdown文件渲染
* server.js 启动内置的本地服务器进行预览`hexo server`
* version.js 显示当前系统版本`hexo version`

只需要几句命令就可以发布文章：
```bash
hexo n #写文章
hexo g #生成
hexo d #部署 # 可与hexo g合并为 hexo d -g
```
至此，先告一段落。