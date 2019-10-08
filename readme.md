## 1. 简介
Angular JS 仿拉勾招聘网完成前端职位展示，职位搜索，登录，收藏职位功能；

## 2. 技术框架
Angular JS
Bower 用来管理js 包
Gulp 3.0 用于编译前端项目代码
Less 

## 3. 开发过程
### 3.1 搭建项目框架
bower 引入第三方依赖
npm 初始化 管理项目
编写gulp file 文件，确定js、html、less/css、image等文件结构;源文件，编译后运行文件，编译后生产文件(压缩出来)。

```gulp
var gulp = require('gulp');
var $ = require('gulp-load-plugins')();
var open = require('open');

var app = {
  srcPath: 'src/',
  devPath: 'build/',
  prdPath: 'dist/'
};

gulp.task('lib', function() {
  gulp.src('bower_components/**/*.js')
  .pipe(gulp.dest(app.devPath + 'vendor'))
  .pipe(gulp.dest(app.prdPath + 'vendor'))
  .pipe($.connect.reload());
});

gulp.task('html', function() {
  gulp.src(app.srcPath + '**/*.html')
  .pipe(gulp.dest(app.devPath))
  .pipe(gulp.dest(app.prdPath))
  .pipe($.connect.reload());
})


gulp.task('json', function() {
  gulp.src(app.srcPath + 'data/**/*.json')
  .pipe(gulp.dest(app.devPath + 'data'))
  .pipe(gulp.dest(app.prdPath + 'data'))
  .pipe($.connect.reload());
});

gulp.task('less', function() {
  gulp.src(app.srcPath + 'style/index.less')
  .pipe($.plumber())
  .pipe($.less())
  .pipe(gulp.dest(app.devPath + 'css'))
  .pipe($.cssmin())
  .pipe(gulp.dest(app.prdPath + 'css'))
  .pipe($.connect.reload());
});

gulp.task('js', function() {
  gulp.src(app.srcPath + 'script/**/*.js')
  .pipe($.plumber())
  .pipe($.concat('index.js'))
  .pipe(gulp.dest(app.devPath + 'js'))
  .pipe($.uglify())
  .pipe(gulp.dest(app.prdPath + 'js'))
  .pipe($.connect.reload());
});

gulp.task('image', function() {
  gulp.src(app.srcPath + 'image/**/*')
  .pipe($.plumber())
  .pipe(gulp.dest(app.devPath + 'image'))
  .pipe($.imagemin())
  .pipe(gulp.dest(app.prdPath + 'image'))
  .pipe($.connect.reload());
});

gulp.task('build', ['image', 'js', 'less', 'lib', 'html', 'json']);

gulp.task('clean', function() {
  gulp.src([app.devPath, app.prdPath])
  .pipe($.clean());
});

gulp.task('serve', ['build'], function() {
  $.connect.server({
    root: [app.devPath],
    livereload: true,
    port: 3000
  });

  open('http://localhost:3000');

  gulp.watch('bower_components/**/*', ['lib']);
  gulp.watch(app.srcPath + '**/*.html', ['html']);
  gulp.watch(app.srcPath + 'data/**/*.json', ['json']);
  gulp.watch(app.srcPath + 'style/**/*.less', ['less']);
  gulp.watch(app.srcPath + 'script/**/*.js', ['js']);
  gulp.watch(app.srcPath + 'image/**/*', ['image']);
});

gulp.task('default', ['serve']);

```



```
bower 第三方依赖管理工具
常用命令：
bower init;  初始化项目
bower install;   安装第三方
bower uninstall ； 卸载第三方依赖
配置文件：
.bowerrc ;  修改安装目录
bower.json 管理第三方依赖
npm i  -g bower
bower install --save angular#1.7.7

null > .bowerrc
文件结构规划
```
.git  
bower_components  
build  
dist  
node_modules  
src  
   data  
   image  
   script  
     config  

```

```

### 3.2 开发思路

单页面应用，主页引入编译后位置的 css 和 JavaScript文件；

``` html
<!DOCTYPE html>
<html lang="en" ng-app="app">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="initial-scale=1,maximum-scale=1,minimum-scale=1,user-scalable=no">
  <title>webapp</title>
  <link rel="stylesheet" href="/css/index.css" media="screen" title="no title" charset="utf-8">
</head>
<body>
  <div ui-view>

  </div>
  <script type="text/javascript">
    document.getElementsByTagName('html')[0].style.fontSize = window.screen.width / 10 + 'px';
  </script>
  <script src="vendor/angular/angular.min.js" charset="utf-8"></script>
  <script src="vendor/angular-ui-router/release/angular-ui-router.min.js" charset="utf-8"></script>
  <script src="vendor/angular-cookies/angular-cookies.min.js" charset="utf-8"></script>
  <script src="vendor/angular-validation/dist/angular-validation.js" charset="utf-8"></script>
  <script src="vendor/angular-animate/angular-animate.min.js" charset="utf-8"></script>
  <script src="js/index.js" charset="utf-8"></script>
</body>
</html>

```

页面展示html 文件按页面文件和模板文件区分；页面文件由当前页面文件和模板组成；对于通用部分功能编写成模板供所有模块使用。

模板，通过自定义指令(directrive)文件、less style 文件、模板HTML 文件完成自定义指令模板开发。

style 文件，通过less编写，将公用颜色、函数抽取出来单独作为变量文件；

```less
//公用颜色
@defaultColor: #fff;
@headBgColor: #12d5b5;
@warnColor: #feae11;
@muteColor: #9f9e9d;
@lineColor: #d4d4d4;
// 公用颜色类
.bg-w {
  background-color: #fff;
}
.c-w {
  color: #fff;
}
.c-r {
  color: red;
}
//公用函数
.ng-enter-active {
  opacity:1;
}

.fs(@px) {
  font-size: unit(@px / 37.5, rem);
}

.w(@px) {
  width: unit(@px / 37.5, rem);
}

.h(@px) {
  height: unit(@px / 37.5, rem);
}

```



```
less与css
定义变量：less可以写将css值设置为变量。@标识变量
样式模块化：可以写函数，供模块使用。

后代选择器：支持嵌套
文件引用:@import "head.less" .引入全局变量；初始化less文件，把要编译的所有less文件都引用进来，一起编译，生成一个css文件。

```

4. 示例源码

   来源于网络。