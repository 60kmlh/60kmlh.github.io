---
title: 使用gulp移动打包的静态资源到java项目
date: 2017-03-25
tags: ['gulp','效率']
categories: ['工具']
---
## 背景
在java项目里使用vue作为前端框架，使用webpack作为打包工具。打包完的静态资源需要移动到java项目的WebContent文件夹里，本来webpack可以指定输出的路径，但由于HtmlWebpackPlugin插件不支持使用jsp作为模板，因此决定引入gulp来实现这个功能。
## 目标
将webpack打包完的js,css，插入到jsp模板里，同时将静态资源移动java项目WebContent里对应的文件夹下面的(这个功能也可以用webpack实现)。
## 实现

### 需要用到的npm包和gulp插件
````javascript
"devDependencies": {
    "del": "^2.2.2",
    "gulp": "^3.9.1",
    "gulp-cheerio": "^0.6.2",
    "gulp-replace": "^0.5.4",
    "through2": "^2.0.3"
  }
````
### 过程

(1)在java项目下的WebContent文件夹下，新建前端源码的文件夹src。

(2)每次开发完成后，打包生成src/dist文件夹。清空WebContent下的资源文件夹，将src/dist中的静态资源js，css，img移动到对应的资源文件夹。
````javascript
//清空文件夹
gulp.task('del',()=>{
    del(['../css/*'],{force:true})//或者使用gulp-clean
    del(['../images/*'],{force:true})
    del(['../scripts/*'],{force:true})
    del(['../WEB-INF/jsp/*'],{force:true})
})
//拷贝文件到指定文件夹
gulp.task('cp', ()=>{
    return gulp.src('src/*/*').pipe(gulp.dest('../'))
})
````
(2)获取src/dist/scripts文件夹下的manifest.js,app.js,vendor.js,app.css的hash值，使用gulp-cheerio插件生成对应的script，link节点，插入模板，并存放入目标文件夹WebContent/WEB-INF/jsp。

WebContentscr/src下的jsp模板
````javascript
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<jsp:include page="common.jsp"></jsp:include>
 <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
 <% 
 	String sid=request.getParameter("sid");
 %>
<c:set var="cpath" value="${pageContext.request.contextPath}"></c:set>
<!DOCTYPE html>
<html>
    <head>
        <meta charset=UTF-8>
        <meta name=viewport content="width=640,user-scalable=no,target-densitydpi=device-dpi">
        <title>project</title>
    </head>
    <body>
        <div id=app></div>
        <input type="text" value="${param.notbindPhone}" style="display: none" id="notbindPhoneId">
        <input type="text" value="${userMobile}" style="display: none" id="mobile">
    </body>
</html>
````
cheerio插件操作html的,并不支持jsp的dom操作,在做dom操作的时候,首先需要把源文件html进行智能补全处理,会把未结束的标签补齐，会使页面的jsp标签乱掉了。解决的办法,在调用cheerio 之前,先把jsp标签的<% 和%>替换掉。这样cheerio 会把他们是当做普通的字符串处理.等cheerio处理结束后,再替换回来。

````javascript
//获取hash值
let hash = {}
gulp.src('src/scripts/manifest.*.js').pipe(through2.obj((file, enc, cb)=>{
   hash.manifest = file.history.toString().replace(file.base,'').split('.')[1]
}))
////向jsp模板插入静态资源
gulp.src('src/index.jsp').pipe(cheerio({run:$=>{
  $('body').append('<script src="../'+jsPath+'/manifest.'+hash.manifest+'.js"></script>')
},parserOptions:{decodeEntities: false}}))
````
WebContent//WEB-INF/jsp下生成的jsp
````javascript
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<jsp:include page="common.jsp"></jsp:include>
 <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
 <% 
 	String sid=request.getParameter("sid");
 %>
<c:set var="cpath" value="${pageContext.request.contextPath}"></c:set>
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=640,user-scalable=no,target-densitydpi=device-dpi">
        <title>project</title>
    <link href="../css/app.e34a801012871ec0a999.css" rel="stylesheet"></head>
    <body>
        <div id="app"></div>
        <input type="text" value="${param.notbindPhone}" style="display: none" id="notbindPhoneId">
        <input type="text" value="${userMobile}" style="display: none" id="mobile">
    <script src="../scripts/manifest.9e7be796d2fe2d9d2c84.js"></script>
    <script src="../scripts/vendor.d806109533dbf074df4e.js"></script>
    <script src="../scripts/app.e34a801012871ec0a999.js"></script>
    </body>
</html>
````

### 完整源码

````javascript
var gulp = require('gulp')
var del = require('del') //清空文件夹
var through2 = require('through2') //读取文件名
var cheerio = require('gulp-cheerio') // 操作jsp模板，若使用gulp-rev会重复添加hash
var replace = require('gulp-replace') //替换文本
var jsPath = 'scripts'

//清空文件夹
gulp.task('del',()=>{
    del(['../css/*'],{force:true})//或者使用gulp-clean
    del(['../images/*'],{force:true})
    del(['../scripts/*'],{force:true})
    del(['../WEB-INF/jsp/*'],{force:true})
})
//拷贝文件到指定文件夹
gulp.task('cp', ()=>{
    return gulp.src('src/*/*').pipe(gulp.dest('../'))
})

//向jsp模板插入静态资源
gulp.task('insert',()=>{
    let hash = {}
    function handleJsp(hash){
        return gulp.src('src/index.jsp')
        .pipe(replace("<%","~%")) //替换掉jsp标签的<%
        .pipe(replace("%>","%~"))//替换掉jsp标签的<%
        .pipe(cheerio({run:$=>{
            $('body').append('<script src="../'+jsPath+'/manifest.'+hash.manifest+'.js"></script>')        
            $('body').append('<script src="../'+jsPath+'/vendor.'+hash.vendor+'.js"></script>')
            $('body').append('<script src="../'+jsPath+'/app.'+hash.app+'.js"></script>')
            $('head').append('<link href="../css/app.'+hash.css+'.css" rel="stylesheet">')
        },parserOptions:{decodeEntities: false}}))//防止汉字被转码
        .pipe(replace("%~","%>")) //替换回来jsp标签的<%
        .pipe(replace("~%","<%"))//替换回来jsp标签的<%
        .pipe(gulp.dest('../WEB-INF/jsp/'))
    }
    //获取hash值
    gulp.src('src/scripts/manifest.*.js').pipe(through2.obj((file, enc, cb)=>{
        hash.manifest = file.history.toString().replace(file.base,'').split('.')[1]
    }))
    gulp.src('src/scripts/vendor.*.js').pipe(through2.obj((file, enc, cb)=>{
        hash.vendor = file.history.toString().replace(file.base,'').split('.')[1]
    }))
    gulp.src('src/scripts/app.*.js').pipe(through2.obj((file, enc, cb)=>{
        hash.app = file.history.toString().replace(file.base,'').split('.')[1]
    }))
    gulp.src('src/css/app.*.css').pipe(through2.obj((file, enc, cb)=>{
        hash.css = file.history.toString().replace(file.base,'').split('.')[1]
    }))
    handleJsp(hash)
})

gulp.task('default',['del','cp','insert'])
````
