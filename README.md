# vue-cli-multipage

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

# vue-cli 搭建多页面

## 命令行工具 (CLI)

Vue.js 提供一个官方命令行工具，可用于快速搭建大型单页应用。该工具提供开箱即用的构建工具配置，带来现代化的前端开发流程。只需几分钟即可创建并启动一个带热重载、保存时静态检查以及可用于生产环境的构建配置的项目：

```
# 全局安装 vue-cli
$ npm install --global vue-cli

# 创建一个基于 webpack 模板的新项目
$ vue init webpack 21dj-mobile
```
![安装过程](http://olgjbx93m.bkt.clouddn.com/WX20170804-01.png)

```
// 公众号端为多页面，所以用不到 vue-router
Install vue-router? No

// ESLint 代码格式化
Use ESLint to lint your code? No

// 单元测试
Setup unit tests with Karma + Mocha? No
Setup e2e tests with Nightwatch? No
```
## 启动项目

```
# 进入项目目录
$ cd 21dj-mobile

# 安装npm
$ npm install

# 运行
$ npm run dev
```

访问 http://localhost:8080/ 就可以看到 Vue 启动页面

![Vue 启动页面](http://olgjbx93m.bkt.clouddn.com/WX20170804-02.png)

## 打包项目

```
npm run build
```
这样就可以在项目目录下生成 dist 文件夹，项目页面及样式都存放在 dist 文件夹中。

![项目目录](http://olgjbx93m.bkt.clouddn.com/WX20170804-03.png)

## 配置多个入口

修改 config/index.js

```
    moduleName: 'views',
```
![config](http://olgjbx93m.bkt.clouddn.com/WX20170805-03.png)

修改 /21dj-mobile/build/webpack.base.conf.js 

```
var entries = utils.getMultiEntry('./src/' + config.moduleName + '/**/**/*.js'); // 获得入口js文件


// entry: {
//   app: './src/main.js'
// },
entry: entries,
```
如图所示：

![如图所示](http://olgjbx93m.bkt.clouddn.com/WX20170805-01.png)

修改 21dj-mobile/build/utils.js , 添加 getMultiEntry 方法

```
var glob = require('glob');


//获取多级的入口文件
exports.getMultiEntry = function(globPath) {
    var entries = {},
        basename, tmp, pathname;

    glob.sync(globPath).forEach(function(entry) {
        basename = path.basename(entry, path.extname(entry));
        tmp = entry.split('/').splice(-4);
        var pathsrc = tmp[0] + '/' + tmp[1];
        if (tmp[0] == 'views') {
            pathsrc = tmp[1];
        }

        //console.log(pathsrc)
        pathname = pathsrc + '/' + basename; // 正确输出js和html的路径
        if (tmp[0] == 'src') {
            pathname = basename; // 正确输出js和html的路径
        }
        entries[pathname] = entry;
        //console.log(pathname + '-----------' + entry);
    });
    return entries;
}
```
代码简单说明：

```
// 给 globPath 赋值
globPath = './src/views/**/**/*.js'

遍历所有的src/views的所有含有.js 文件的文件夹

// 将 entry 按照 '/' 分割成数组，并获取数组的后四个
tmp = entry.split('/').splice(-4);
```

## 开发环境-多页面
修改 21dj-mobile/build/webpack.dev.conf.js

```
//注释 HtmlWebpackPlugin

// new HtmlWebpackPlugin({
//   filename: 'index.html',
//   template: 'index.html',
//   inject: true
// }),
```
添加遍历html方法，添加多个 HtmlWebpackPlugin

```
var pages = utils.getMultiEntry('./src/' + config.moduleName + '/**/**/*.html');
for (var pathname in pages) {
    // 配置生成的html文件，定义路径等
    var conf = {
        filename: pathname + '.html',
        template: pages[pathname], // 模板路径
        chunks: [pathname], // 每个html引用的js模块
        inject: true // js插入位置
    };
    // 需要生成几个html文件，就配置几个HtmlWebpackPlugin对象
    module.exports.plugins.push(new HtmlWebpackPlugin(conf));
}
```

![代码修改](http://olgjbx93m.bkt.clouddn.com/WX20170805-02.png)

在 src 文件下 新建多个文件夹，如图所示：

![文件夹](http://olgjbx93m.bkt.clouddn.com/WX20170805-04.png)

运行 dev 命令：

```
npm run dev
```
可以访问 
http://localhost:8080/
http://localhost:8080/login.html
http://localhost:8080/home/list.html
http://localhost:8080/iconfont/list.html

![运行结果](http://olgjbx93m.bkt.clouddn.com/WX20170805-05.png)

## 生产环境-多页面
修改 21dj-mobile/build/webpack.prod.conf.js

```
var entries = utils.getMultiEntry('./src/' + config.moduleName + '/**/**/*.js'); // 获得入口js文件
var chunks = Object.keys(entries);
```
注释掉 HtmlWebpackPlugin 和 CommonsChunkPlugin，添加自定义的 CommonsChunkPlugin 和 HtmlWebpackPlugin方法

```
new webpack.optimize.CommonsChunkPlugin({
        name: 'vendor',
        chunks: chunks,
        minChunks: 4 || chunks.length
    }),
```

```
//构建生成多页面的HtmlWebpackPlugin配置，主要是循环生成
var pages = utils.getMultiEntry('./src/' + config.moduleName + '/**/**/*.html');
for (var pathname in pages) {

    var conf = {
        filename: pathname + '.html',
        template: pages[pathname], // 模板路径
        chunks: ['vendor', pathname], // 每个html引用的js模块
        inject: true, // js插入位置
        hash: true
    };

    webpackConfig.plugins.push(new HtmlWebpackPlugin(conf));
}
```

运行 npm run build ，打包项目，运行结果：

![运行结果](http://olgjbx93m.bkt.clouddn.com/WX20170805-06.png)

## 参考资料

vue2-cli-vux2-multe-page ： https://github.com/bluefox1688/vue-cli-multi-page

vue-cli + webpack 多页面实例应用：
http://www.cnblogs.com/fengyuqing/p/vue_cli_webpack.html


