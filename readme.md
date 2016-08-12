#autowebapp

>前端自动化,工程化模板

[TOC]

## 主要技术

- **nodejs** 
- **npm**  [官网](https://www.npmjs.com/) 查找各种库很方便
- **cnpm** 淘宝开发的npm镜像 
- **bower** 包管理工具 基本用法 `bower install jquery --save` `bower list` `bower update`
- **gulp**  自动化发布 `cnpm install gulp` `gulp task` 运行gulp任务

## 主要能力

- less 转成css ,打包css ,合并多个css文件并压缩
- 模块化编程的js文件打包合并和压缩
- 调试自动刷新页面
- 图片合并成雪碧图,并自动生成css 自己调用相应的class 

## 使用方法

`git clone https://github.com/lihongbin100/autowebapp.git`

## 重要文件

gulpfile.js

``` javascript

//引入插件

var gulp = require('gulp'),
    connect = require('gulp-connect'),//自动刷新页面组件
    less = require('gulp-less'),//less转化为CSS组件
    cssmin = require('gulp-clean-css'),//css压缩组件
    rename = require('gulp-rename'),//文件重命名组件
    sourcemaps = require('gulp-sourcemaps'),//生成map文件
    uglify = require('gulp-uglify'),//js压缩组件
    concat = require('gulp-concat'),//多文件合并组件
    webpack = require('gulp-webpack'),//webpack 打包模块化编程的js
    spritesmith = require('gulp.spritesmith');//图片合并成雪碧图

var app = {
    appName: "gcks-heatmap",
    lessPath: "./app/less/",
    jsPath: "./app/js/",
    imagePath: "./app/images/",
    binPath: "./bin/"
};

gulp.task('lessToCssAndMin', function () {
    gulp.src([app.lessPath + '*.less'])
        .pipe(sourcemaps.init())
        .pipe(less())
        .pipe(cssmin()) //兼容IE7及以下需设置compatibility属性 .pipe(cssmin({compatibility: 'ie7'}))
        .pipe(sourcemaps.write())
        .pipe(rename(function (path) {
            path.basename = app.appName + ".min";
        }))
        .pipe(gulp.dest(app.binPath + 'css'));
});

gulp.task('jsMin', function () {
    gulp.src([app.jsPath + '*.js'])
        .pipe(webpack())
        .pipe(uglify(
            {
                mangle: false,//类型：Boolean 默认：true 是否修改变量名
                compress: false,//类型：Boolean 默认：true 是否完全压缩
                preserveComments: 'no' //保留所有注释
            }))
        .pipe(concat(app.appName + ".min.js"))
        .pipe(connect.reload())
        .pipe(gulp.dest(app.binPath + 'js'));
});

var icon="icon-1.png";

gulp.task('imagesMin', function () {
    return gulp.src('./app/images/icon-*/*.png')//需要合并的图片地址
        .pipe(spritesmith({
            imgName: icon,//保存合并后图片的地址
            cssName: '../css/icon.css',//保存合并后对于css样式的地址
            padding: 5,//合并时两个图片的间距
            algorithm: 'binary-tree',//注释1
            cssTemplate:
                function (data) {
                    // console.log(data);
                    var arr = [];
                    data.sprites.forEach(function (sprite) {
                        // console.log(sprite);
                        arr.push("." + sprite.name +
                            "{" +
                            "background-image: url('../images/"+icon+"');" +
                            "background-position: " + sprite.px.offset_x  +" "+ sprite.px.offset_y + ";" +
                            "width:" + sprite.px.width + ";" +
                            "height:" + sprite.px.height + ";" +
                            "}\n");
                    });
                    return arr.join("");
                }
        }))
        .pipe(gulp.dest(app.binPath + 'images'));
});

//使用connect启动一个Web服务器
gulp.task('connect', function () {
    connect.server({
        root: './bin',
        livereload: true
    });
});

gulp.task('html', function () {
    gulp.src('./app/*.html')
        .pipe(connect.reload())
        .pipe(gulp.dest(app.binPath));
});

//创建watch任务去检测html文件,其定义了当html改动之后，去调用一个Gulp的Task
gulp.task('htmlWatch', function () {
    gulp.watch(['./app/*.html'], ['html']);
});

gulp.task('lessWatch', function () {
    gulp.watch(app.lessPath + '*.less', ['lessToCssAndMin']); //当所有less文件发生改变时，lessToCssAndMin
});

gulp.task('jsWatch', function () {
    gulp.watch(app.jsPath + '*.js', ['jsMin']); //当所有less文件发生改变时，jsMin
});

//运行Gulp时，默认的Task
gulp.task('default', ['connect', 'htmlWatch', 'lessWatch', 'jsWatch']);

```

