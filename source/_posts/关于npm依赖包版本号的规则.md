---
title: 关于npm依赖包版本号的规则
date: 2017-08-05
tags: ['npm','版本']
categories: ['工具']
---
之前有新同事接手项目，npm包安装完后，发现有功能失效了，经过排查后发现，有个依赖包安装完后的版本和package.json写的不一样。同时包的api有大的变动，导致了问题的发生。package.json为什么没有“生效”，带着疑惑，在官网上面找到了答案。
<!--more-->
# semantic versioning
根据官网的描述，npm包的版本采用semantic versioning，即语义化版本。

研究了semantic versioning的[规范](https://docs.npmjs.com/misc/semver)后，找到了问题产生的原因。

1. semantic versioning的命名规则：
> 版本格式：主版本号.次版本号.修订号，
> 
> 版本号递增规则如下：
> 
> 主版本号：当你做了不兼容的 API 修改， 
> 
> 次版本号：当你做了向下兼容的功能性新增，
> 
> 修订号：当你做了向下兼容的问题修正。
> 
> 先行版本号及版本编译信息可以加到“主版本号.次版本号.修订号”的后面，作为延伸。
2. 高级范围语法

以^开头的版本号规则叫做Caret Ranges，官网是这样描述Caret Ranges的：
> Allows changes that do not modify the left-most non-zero digit in the [major, minor, patch] tuple. In other words, this allows patch and minor updates for versions 1.0.0 and above, patch updates for versions 0.X >=0.1.0, and no updates for versions 0.0.X.

也就是说^开头的版本号， 不允许对[主版本号, 次版本号, 修订号]元组中最左边非零数字的更改，其他两位是可以更改的。

以~开头的版本号规则叫做Tilde Ranges，Tilde Ranges的规则如下：
> Allows patch-level changes if a minor version is specified on the comparator. Allows minor-level changes if not.

和Caret Ranges相比，Tile Ranges只允许更改[主版本号, 次版本号, 修订号]的最后一位，即修订号。

例子如下：

^1.2.1 := >=1.2.1 <2.0.0

~1.2.1 :=  >=1.2.1 <1.3.0

而当运行npm i --save xx的时候，npm会优先考虑Caret Ranges，package.json里面的依赖包版本号就是^开头的。

当包的次版本号或修订号有更新的时候，执行npm i安装下来的就是符合规则的最新的包了，而不一定是正确安装的版本。

关于之前的问题，更新小版本的同时，包的api又发生了变动，才导致了问题的发生。

semantic versioning的规范里有一条是这样的：
> Major version X (X.y.z | X > 0) MUST be incremented if any backwards incompatible changes are introduced to the public API. 

主版本号 X（X.y.z | X > 0）必须在有任何不兼容的修改被加入公共 API 时递增。

也就是说之前出问题的包，如果严格遵循了semantic versioning,在api有break change是主版本号进行递增，是不会升级到api不兼容的版本，导致功能失效的。
# 解决办法
一般默认npm安装使用^开头的版本号是比较好的选择，可以接受小版本和补丁版本的变化，保证包中的小bug可以得到修复，使用~版本更改会更小。

也可以指定特定的版本号，直接写1.1.2，前面不加~^或其它符号，这样就是写死了版本，但是如果依赖包发布新版本修复了一些小bug，则需要手动取更改package.json文件里对应的版本号。
# 参考资料
1. [https://docs.npmjs.com/misc/semver](https://docs.npmjs.com/misc/semver)
2. [http://semver.org](http://semver.org)
