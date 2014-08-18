---
layout: post
title: 领略SCSS的魅力
date: 2014-07-19 13:14:35 +0800
comments: true
categories: [css sass]
---
在今天的移动互联网环境下，高效敏捷地开发成为产品成功与否的重要因素。开发效率和代码可重构性，可重用性，成为衡量开发者能力与项目好坏的重要指标。在这种环境下，传统的CSS由于种种原因，代码沉余，难以维护，随着项目增大，问题变得突出，开发效率开始被拖低，严重者影响商业模式的运转。因此前端开发者需要一种或一系列方法或机制，来维护CSS代码，从而提高开发效率和项目代码质量。
### 初识SCSS
SCSS全名叫做Sassy CSS，前身是Sass，意思为语法非凡的CSS。具有很多普通CSS不具有的特性。
#### 变量
``` scss
$tint-color: rgb(0, 122, 255);
$font-stack: HelveticaNeue, Arial, sans-serif;

a {
    color: $tint-color;
    font: $font-stack;
}

p {
    font: $font-stack;
}

```
在前端开发中，网页往往有主色调。很多不同的部分使用同一种颜色值，比如超链接，logo，装饰的颜色都一样。如果设计师对此色值进行微调，在传统的CSS中，需要使用编辑器或Unix命令查找替换所有色值。而在SCSS中，仅需要修改一个变量，就可以达到目的。
#### 嵌套
``` scss
.cell {
    border: $some-border;
    .indicator {
        color: $tint-color;
    }
    .icon {
        top: $top-margin;
        left: $left-margin;
    }
}
```
在很多情况下，一个元素或一个类只有在某一元素内才有意义，比如模仿iOS的标准表格视图，一旦图标出现在表格之外便没有意义。使用嵌套，不仅能够令css的层级与html相同，更有助于释放命名空间，减少奇怪bug发生和难debug的可能性。
#### 片段和引入
``` scss
/* _base.scss */

$text-color: #333333;
$bg-color: white;
$tint-color: rgb(0, 122, 255);

* {
    box-sizing: border-box;
}

body {
    color: $text-color;
    background-color: $bg-color;
    margin: 0px;
    font: {
        family: $font-stack;
        size: $body-font-size;
        weight: normal;
        style: normal;
    }
}

/* some_view.scss */
@import 'base';

a {
    color: $tint-color;
}

/* another_view.scss */
@import 'base';

```
不同视图可能会有相同的基本样式，比如主色调或者一些尺寸值。可以将相同的内容一并放入片段文件，来避免写一个巨大的没有命名空间的全局样式表。SCSS足够聪明理解你的@import究竟是引入SCSS的还是保留在CSS中的。
#### 混合
``` scss
/* 浏览器兼容示例 */
@mixin user-select($selectable) {
    -webkit-touch-callout: $selectable;
    -webkit-user-select: $selectable;
    -khtml-user-select: $selectable;
    -moz-user-select: $selectable;
    -ms-user-select: $selectable;
    user-select: $selectable;
}
.cell { @include user-select(none); }

/* 手机下的超链接示例 */
@mixin tap-highlight-color($color) {
    -webkit-tap-highlight-color: $color;
}
@mixin a-color($color, $highlight-color) {
    color: $color;
    cursor: pointer;
    text-decoration: none;
    @include tap-highlight-color($highlight-color);
}
@mixin a-color-set($color, $highlight-color) {
    @include a-color($color, $highlight-color);
    &:link &:visited &:hover &:active {
        @include a-color($color, $highlight-color);
    }
}
a {
    @include a-color-set($normal-color, $tap-highlight-color);
}
```
由于浏览器的兼容性原因，有时不得不对同一个样式用不同的浏览器内核前缀写多次，混合很优雅地解决了这个问题。混合的强大之处还表现在可以传递参数。
#### 扩展和继承
``` scss
.cell {
    /* 普通单元格的属性 */
}
.editing-cell {
    @extend .cell;
    /* 可编辑的单元格拥有更多属性，更像是子类继承了父类 */
}
```
继承令开发者不需要给一个元素赋值两个类。也使代码清晰，干净。当一个元素与另外一个元素有相似之处，就可以使用继承，这样修改“父类”，“子类”也会随之改变。
### 配置SCSS环境
现在你已经可以快乐地写SCSS代码了，但是还不清楚写好的SCSS代码如何放到网页中去。
  
SCSS需要Ruby环境，OS X系统下预装了Ruby和相关命令行工具。如果您没有Ruby，或者想更新Ruby的版本，请移步：[Ruby官方的下载页面](https://www.ruby-lang.org/en/downloads/)
  
安装SCSS很容易
``` sh
gem install sass
```

将SCSS转CSS
``` sh
sass input.scss output.css
```
在最新的Rails项目中，可以直接使用SCSS。
### SCSS进阶用法
#### 注释
SCSS支持两种注释，/\* \*/可以被编译到CSS中，//则会被省略。
#### Sass脚本
``` bash
sass -i
```
使用sass -i命令来启动交互shell。
Sass支持数字，文本，颜色，布尔，空，列表，哈希表这些数据类型。支持<, >, <=, >=, ==, !=等运算符。有大量内建函数，同时支持自定义函数。
  
详细解释如下
``` scss
$number: 1em; // 数字，数字有单位，比如1px, 20%, 1vh
$string: center; // 文本，使用的引号会被CSS保留，字符串插值时除外
$color: white; // 颜色，也可以是#FFFFFF, rgb(255,255,255)
$boolean: true; // 布尔
$null: null; // 空
$list: 20px 30px 40px 50px; // 列表，有一种是空格隔开，另一种是逗号隔开，比如Helvetica, Arial, sans-serif
$map: (key1: value1, key2: value2, key3: value3); // 哈希表
p {
    color: #010203 + #040506;
    font: #{$font-size}/#{$line-height}; // 除号在直接写的时候不进行运算，使用变量时，可以使用插值来避免运算。
}
$red: red;
$translucent-red: transparentize($red, 0.2); // 使用函数
// 在写框架的时候，运算符和函数非常有用。
```
### @debug 和 @warn
使用@debug和@warn，可以在编译的时候输出一些调试信息和警告信息。
``` scss
@debug 10em + 12em;
```
``` scss
@if unitless($x) {
    @warn "Assuming #{$x} to be in pixels";
    $x: 1px * $x;
}
```
暂时写到这么多，更详细的在[官方的文档](http://sass-lang.com/documentation/file.SASS_REFERENCE.html)。所有的内建函数列表在[这里](http://sass-lang.com/documentation/Sass/Script/Functions.html)。

关于SCSS，欢迎大家和我一起讨论。
