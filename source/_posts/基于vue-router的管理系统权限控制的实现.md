---
title: 基于vue-router的管理系统权限控制的实现
date: 2017-07-08
tags: ['vue','vue-router','管理系统','权限控制']
categories: ['工作总结']
---
# 背景
之前使用vue做的后台管理系统，具有账号体系和权限管理的功能，前端需要实现根据权限异步生成菜单，根据用户权限来控制页面和相关按钮的显示，以及路由控制，这里主要讨论解决路由的问题。
# 解决思路
## 思路一: 写出全部的路由配置，利用导航钩子控制无权限页面的跳转。

登陆完之后，获取用户权限的相关信息存入vuex，相关按钮根据权限信息控制显示或隐藏。

写死所有的路由配置，包括404页的跳转，输入不匹配的地址都跳到404页面。

在路由的meta信息里存储权限信息，可以是一个权限码，具体看和后台的约定。
````javascript
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/hello',
      name: 'Hello',
      component: resolve => require(['@/components/FirstPage'], resolve),
      meta: {
      permission:'firstpage'
      },
      children: [
        {
          path:'child',
          name: 'firstPageChild',
          component: resolve => require(['@/components/Child'], resolve),
          meta: {
      permission:'firstpagechild'
      } 
        }
      ]
    },
    {

      path:'*',
      name:'notFound',
      component: resolve => require(['@/components/NotFound'], resolve)
    }//404路由
  ]
})

````
此时对于无权限的页面，隐藏按钮，用户无对应的入口按钮，是无法通过点击对应按钮进入的，但是直接在地址栏输入地址还可以进入。

这里借助vue-router的beforeEach钩子，在进入页面之前，通过路由的meta信息进行权限判断，若无权限则跳到404页面，有权限则正常进行跳转。
````javascript
import router from './router'

router.beforeEach((to, from, next) => {
  console.log(to.meta.permission);
  if(to.matched.forEach(record => {return permissionCode.indexOf(record.meta.permission) == -1})){
  next('/notFound')
  }else{
    next()
  }
})//permissionCode为后台传过来的拥有的权限列表，按具体项目判断方法不一样
````
这里如果是直接打印出to的meta信息，只能获取到当前路由嵌套最深的那一层路由meta信息。

vue-router官方推荐使用matched属性，to.matched数组包含了匹配到的父路由记录以及子路由记录，遍历即可。

如果没有权限则调用next('/notFound')跳转到404页面。有权限则继续调用next()，这里确保要调用next方法。

这个方法的缺点是要将所有的路由和对应权限都写出来。

<br>

## 思路二: 借助addRoutes动态添加路由

vue-router在2.2.0版本新增了router.addRoutes实例方法，用来动态添加路由。

因此可借助这个方法，在用户登陆完之后，获取权限信息，动态生成对应的有权限的路由，之前先写好404路由。
````javascript
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path:'*',
      name:'notFound',
      component: resolve => require(['@/components/NotFound'], resolve)
    }//404路由
  ]
})
````

假设以下为和后台约定好的用来构造路由的权限信息，实际情况可能不一样。
````javascript
 route:[{
    route:'/a',//路由地址
    name: 'A',//路由名称
    component: 'pageA/Main',//组件名称
    meta: {permission: 'A'},//meta信息
    children: [{
        route:'child1',
        name: 'A_A',
        component: 'pageA/ModuleA'
    },{
        route:'child2',
        name: 'A_B',
        component: 'pageA/ModuleB'
    }]
},
{
    route:'/b',
    name: 'B',
    component: 'pageB/Main',
    meta: {permission: 'B'},
    children: [{
        route:'child1',
        name: 'B_A',
        component: 'pageB/ModuleA'
    },{
        route:'child2',
        name: 'B_B',
        component: 'pageB/ModuleB'
    }]
},
{
    route:'/c',
    name: 'C',
    component: 'pageC/Main',
    meta: {permission: 'C'},
    children: [{
        route:'child1',
        name: 'C_A',
        component: 'pageC/ModuleA'
    },{
        route:'child2',
        name: 'C_B',
        component: 'pageC/ModuleB'
    }]
}]
````
将后台传过来的权限信息，配置成符合 routes 选项要求的数组。
````javascript
createRouter(){
      var rootRoute = []
      this.route.forEach((value1, index1) => {
        let parentRoute = {
          path: value1.route,
          name: value1.name,
          component: resolve => require(['./views/'+value1.component], resolve),
          children: [],
          meta: value1.meta
        }

        value1.children.forEach((value2, index2) => {
          let childRoute = {
            path: value2.route,
            name: value2.name,
            component: resolve => require(['./views/'+value2.component], resolve)
          }
          parentRoute.children.push(childRoute)
        })

        rootRoute.push(parentRoute)
      })
      this.$router.addRoutes(rootRoute)
    }//页面级别的组件都放在views文件夹下，Main为主入口，子页面也在同一文件夹
````

这样输入的无权限地址都会不匹配，跳到404路由。

这种方法存在缺点，使用webpack实现按需加载，打包只能打包到主页面级别，例如上面打包出来就是A.js，B.js，C.js，子级页面的代码也被打包进去了，不能再打包一层，这个问题待解决。
