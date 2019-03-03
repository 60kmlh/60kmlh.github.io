---
title: 使用gulp为静态资源添加版本号
date: 2017-05-20
tags: ['gulp','效率']
categories: ['工具']
---
##　背景
日常使用gulp，使用的比较多的功能是为静态资源文件添加版本号，实现在强缓存下的静态资源缓存命中，增量更新。
<!--more-->
在这里记录一下实现过程。
## gulp插件
1. gulp-rev 

将静态资源的命名后面添加上文件指纹hash的值，插件是根据文件内容生成hash，所以可以实现增量更新。
2. gulp-rev-collector 

比较命名变化前后的资源文件名，生成rev-manifest.json文件名对照映射。
3. gulp-clean 

每次执行前，用来清除旧的文件。
4. run-sequence
控制gulp任务的执行顺序。
## 过程
### 定义路径
````javascript
var path = 'src'// 源文件夹
var dist = 'dist'// 目标文件夹
// 定义css、js源文件路径
var cssSrc = path + '/css/*.css',
    jsSrc = path + '/js/*.js',
    imgSrc = path + '/img/*.*'
````
### 添加hash值，生成文件名对照映射
````javascript
// 处理图片
gulp.task('revImg', function(){
    return gulp.src(imgSrc)// 源文件
        .pipe(rev())// 为文件名添加hash值
        .pipe(gulp.dest(dist+'/img'))// 存放到目标文件夹
        .pipe(rev.manifest())// 生成rev-manifest.json文件名对照映射
        .pipe(gulp.dest('rev/img')) // 将rev-manifest.json存放到rev文件夹
});
// 处理css
gulp.task('revCss', function(){
  return gulp.src(['rev/img/*.json', cssSrc])
    .pipe(rev())
    .pipe(revCollector())// 使用revImg任务生成的rev-manifest.json，替换css里面的图片文件名。
    .pipe(gulp.dest(dist+'/css'))
    .pipe(rev.manifest())
    .pipe(gulp.dest('rev/css'))
})
// 处理js
gulp.task('revJs', function(){
  return gulp.src(jsSrc)
    .pipe(rev())
    .pipe(gulp.dest(dist+'/js'))
    .pipe(rev.manifest())
    .pipe(gulp.dest('rev/js'))
})
````
### 更新html文件里的图片，js，css文件名。
````javascript
gulp.task('revHtml', function () {
  return gulp.src(['rev/**/*.json', 'src/*.html'])
    .pipe(revCollector())// 依照rev-manifest.json里的文件名映射进行替换
    .pipe(gulp.dest(dist))
})
````
### 定义主任务，控制执行顺序
````javascript
gulp.task('default', function (done) {
  condition = false;
  runSequence(
    ['clean'],
    ['revImg'],
    ['revCss'],// revCss必须在revImg之后执行，因为依赖img/rev-manifest.json替换css里面的图片名。
    ['revJs'],
    ['revHtml'],
    done);
});
````
## 结果
````javascript
│  gulpfile.js
│  package.json
│
├─dist
│  │  index.html
│  │
│  ├─css
│  │      style-12d9df9be6.css
│  │
│  ├─img
│  │      imgA-67ada573aa.png
│  │      imgB-efc888043f.png
│  │      imgC-3ba7a2eca0.png
│  │
│  └─js
│          index-fd3859f5fc.js
│
├─rev
│  ├─css
│  │      rev-manifest.json
│  │
│  ├─img
│  │      rev-manifest.json
│  │
│  └─js
│          rev-manifest.json
│
└─src
    │  index.html
    │
    ├─css
    │      style.css
    │
    ├─img
    │      imgA.png
    │      imgB.png
    │      imgC.png
    │
    └─js
            index.js
````
css文件和html里面的资源文件名也对应改变。

index.html
````html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <link rel="stylesheet" href="./css/style-12d9df9be6.css">
    <script src='./js/index-fd3859f5fc.js'></script>
</head>
<body>
    <p>title</p>
    <div>
        <img src="./img/imgA-67ada573aa.png" alt="">
        <img src="./img/imgB-efc888043f.png" alt="">
        <img src="./img/imgC-3ba7a2eca0.png" alt="">
    </div>
</body>
</html>
````
style.css
````css
p,span {
    color: grey;
    background-image: url('../img/imgA-67ada573aa.png')
}
````
## 完整代码
````javascript
// gulpfile.js
//引入gulp和gulp插件
var gulp = require('gulp'),
  runSequence = require('run-sequence'),
  rev = require('gulp-rev'),
  revCollector = require('gulp-rev-collector'),
  clean = require('gulp-clean');

var path = 'src'// 源文件夹
var dist = 'dist'// 目标文件夹
// 定义css、js源文件路径
var cssSrc = path + '/css/*.css',
  jsSrc = path + '/js/*.js',
  imgSrc = path + '/img/*.*';

// CSS生成文件hash编码并生成 rev-manifest.json文件名对照映射
gulp.task('revCss', function(){
  return gulp.src(['rev/img/*.json', cssSrc])
    .pipe(rev())
    .pipe(revCollector())
    .pipe(gulp.dest(dist+'/css'))
    .pipe(rev.manifest())
    .pipe(gulp.dest('rev/css'));
});

// 处理图片
gulp.task('revImg', function(){
    return gulp.src(imgSrc)
        .pipe(rev())
        .pipe(gulp.dest(dist+'/img'))
        .pipe(rev.manifest())
        .pipe(gulp.dest('rev/img'));
});

// js生成文件hash编码并生成 rev-manifest.json文件名对照映射
gulp.task('revJs', function(){
  return gulp.src(jsSrc)
    .pipe(rev())
    .pipe(gulp.dest(dist+'/js'))
    .pipe(rev.manifest())
    .pipe(gulp.dest('rev/js'));
});


// Html替换css、js文件版本
gulp.task('revHtml', function () {
  return gulp.src(['rev/**/*.json', 'src/*.html'])
    .pipe(revCollector())
    .pipe(gulp.dest(dist));
});

gulp.task('clean', function () {
    return gulp.src(dist+'/*/*', {read: false})
        .pipe(clean())
})

// 开发构建
gulp.task('default', function (done) {
  condition = false;
  runSequence(
    ['clean'],
    ['revImg'],
    ['revCss'],
    ['revJs'],
    ['revHtml'],
    done);
});
````