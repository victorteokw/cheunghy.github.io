---
layout: post
title: "Slim 模版语言"
date: 2014-08-17 15:23:29 +0800
comments: true
categories: html ruby slim
---
Slim是一个快速的，轻量级的模版语言，它的目标是简化视图语法到仅剩不可或缺的部分，却又不要变的晦涩。
  
Slim的核心语法是这样创造来的：“最少需要什么，什么可以省略？”
  
Slim项目的初衷是极简主义的语法和最高的执行效率，如果一个人不使用Slim，决不是因为速度。Benchmarks在[这里](http://travis-ci.org/slim-template/slim)。
  
### 基本语法
``` ruby
doctype 5
html
  head
    title Slim Example
    meta name="keywords" content="template language"
    meta name="author" content=author
    = csrf_meta_tags
    coffee:
      alert "Slim supports embedded javascript!"
  body
    h1 Markup examples
    #content
      p This example shows you how a basic Slim file looks.
    p
      | file names in #{@dir}
    ul
      - for file in @files
        li
          = file
    div#footer
      == render 'footer'
      | Copyright &copy; #{@year} #{@author}
```
  
### 主要功能
* 优雅的语法
  + 简洁的语法，使用锁进，不带HTML标签
  + 嵌入带标签的HTML
  + 可以配置的缩写，比如#生成\<div id="..."\>，.生成\<div class="..."\>
* 安全性
  + 自动引用特殊字符
  + 支持Rails的 html_safe?
* 可配置，可以使用插件来扩展
  + 支持扩展语法
  + 支持国际化
* 性能更佳
  + 比Rails默认的erb模版性能更好
  + 流支持
* 主流框架支持Slim （Rails，Sinatra，Python有Plim克隆）
* Unicode支持
* 可以嵌入Markdown，Textile等引擎
  
### 令它工作
#### 获得Slim
作为一个ruby gem安装：
```
gem install slim
```
#### 集成到Rails项目中
在Gemfile中添加这行
``` ruby
gem 'slim-rails'
```
#### 将Slim设置成默认的模版
在application.rb的Rails::Application的子类中添加这块
``` ruby
config.generators do |g|
  g.template_engine :slim
end
```
现在，每次创建新的控制器和视图，slim就在那里。
  
### 语法详解
#### 凭缩进来嵌套
缩进深度按照喜好，2个空格，或是5个空格，至少一个空格就可以。
#### | 管道字符
``` ruby
body
  p
    |
      这里有一行字.
```
管道字符告诉Slim，复制这一行到目标html中去。（注意管道字符后面的每一行都要缩进一个深度，多的空格会被直接带到目标html中）这样可以逃避自动引用特殊字符。所以在管道字符后面插入html成为了可能，像这样：
``` ruby
- articles.each do |a|
  | <tr><td>#{a.name}</td><td>#{a.description}</td></tr>
```
#### 流程控制和内容输出
上面一段出现了减号。减号意味着流程控制，常见的流程控制有循环和条件。end是不需要有的，甚至在Slim中是禁止的，因为块是由缩紧决定的。如果Ruby代码太长，可以用友好的\来分隔，参数之间有逗号的，可以省略\。
  
等号告诉Slim制造输出。如果一行代码太长，\的规则一样适用。
``` ruby
= javascript_include_tag 'application',
  "data-turbolinks-track" => true
```
两个等号同样可以，区别是两个等号的不会被自用引用特殊字符。像这样：
``` ruby
p
  ' file names in
  = "<#@dir>"
```
网页上面显示 file names in \</Users/tieh/example\>
生成的html \<p\>file names in &lt;/Users/tieh/example&gt;\</p\>
``` ruby
p ' file names in
  == "<#@dir>"
```
网页上面显示 file names in， 后面的字却不见了，然后看下源代码
生成的html是 \<p\>file names in \</Users/1/work/demo\>\</p\>，一个奇怪的标签。
总结一下，生成内容的话，尽量使用=，如果生成html标签的话，使用==。
这段代码中出现一个单引号，单引号代替管道字符是为了留出一个右空格，类似的用法还有=>，=<，==>，==<，tag<，tag>，tag<>，和你想象的一样。
  
#### 代码注释
``` ruby
body
  p
    | 我不是注释
    / 我是Slim注释
      我也是Slim注释
    /! 我是html注释
```
注释使用/斜杠，单个斜杠不会被复制到目标html中，带有叹号的会被复制到html中，上面一段slim代码会生成这样的html代码：
``` html
<body><p><!--我是html注释--></p></body>
```
##### 特殊照顾IE
``` ruby
/[if IE]
  p 换一个好点的浏览器
```
上面这段会被生成
``` html
<!--[if IE]><p>换一个好点的浏览器</p><![endif]-->
```
  
#### 文档声明
文档声明标签用来轻松地生成复杂的文档声明。一般在Slim的最上面。
``` ruby
doctype 5
/ 会生成<!DOCTYPE html>
doctype html
/ 同上
doctype 1.1
/ 生成xhtml的<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN"
    "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
doctype xml
/ 生成xml的声明<?xml version="1.0" encoding="utf-8" ?>
```
多数可能会用不到，并不多做解释。它会按照你预想的来工作，比如doctype frameset，doctype strict，doctype basic，doctype transitional等。详细列表在[这里](http://rdoc.info/gems/slim/frames)
  
#### 关闭标签
关闭标签无害，你可以这样写，但不必要，因为标签会被自动关闭。
``` ruby
img src="image.png" /
```
  
#### 行内嵌套
你可以使用行内嵌套来让代码紧凑。
``` ruby
ul
  li.first: a href="/a" A link
  li: a href="/b" B link
```
  
#### 文本上下文
文本可以与属性写在同一行。
``` ruby
body
  h1 id="headline" Welcome to my site.
```
文本也可以被嵌套，但必须用一个管道字符来取消处理。
``` ruby
body
  h1 id="headline"
    | Welcome to my site.
```
  
#### 动态内容
动态内容一样使用等号和双等号，用法和你预期的一样。动态内容可以写在同一行，也可以嵌套。
``` ruby
body
  h1 id="headline" = page_headline
/ 两者都可以
body
  h1 id="headline"
    = page_headline
```
  
#### 属性
属性的写法就像去掉开闭标签的html。可以直接写：
``` ruby
img src="path/to/image.png"
```
也可以使用动态的属性：
``` ruby
img src=my_rails_helper_method
```
属性可以被封装，来增强代码可读性：
``` ruby
body
  h1(id="logo") = page_logo
  h2[id=="tagline"
    class="small tagline"] = page_tagline
```
被封装的属性，可以折行。你可以自己定义封装的符号。默认的有()，[]，{}。两个等号同样可以取消被转换处理。
  
#### 字符串插值
在写文本的地方，比如双引号之中，和html的标签中的文本，你可以使用字符串插值，在标签中的文本里，#@instance_var不会被处理，可以不简略地写#{@instance_var}。
``` ruby
body
  table
    - for user in users
      td id="user_#{user.id}" class=user.role
        a href=user_action(user, :edit) Edit #{user.name}
        a href=(path_to_user user) = user.name
```
  
#### 布尔属性
true, false, nil会被解释为布尔。赋值可以不带引号。更进一步地（简化），如果使用属性封装，可以省略赋值。
``` ruby
input type="text" disabled="disabled"
input type="text" disabled=true
input(type="text" disabled)

input type="text"
input type="text" disabled=false
input type="text" disabled=nil
```
  
#### 属性混合
标签和属性是可以混合在一起写的，并且可以配置。
``` ruby
a.menu class="highlight" href="http://slim-lang.com/" Slim-lang.com
```
会生成
``` html
<a class="menu highlight" href="http://slim-lang.com/">Slim-lang.com</a>
```
  
#### 展开的属性
展开缩写允许你把ruby hash转换为一对对属性内容
``` ruby
.card*{'data-url'=>place_path(place), 'data-id'=>place.id} = place.name
```
会生成
``` ruby
<div class="card" data-id="1234" data-url="/place/1234">Slim's house</div>
```
你可以使用返回hash的语句。
``` ruby
.card *method_which_returns_hash = place.name
.card *@hash_instance_variable = place.name
```
  
#### 动态标签
你可以凭展开缩写来做到动态标签，只需要一个tag键。
``` ruby
ruby:
  def a_unless_current
    @page_current ? {:tag => 'span'} : {:tag => 'a', :href => 'http://slim-lang.com/'}
  end
- @page_current = true
  *a_unless_current Link
- @page_current = false
  *a_unless_current Link
```
会生成
``` html
<span>Link</span><a href="http://slim-lang.com/">Link</a>
```

#### 嵌入的引擎
像这样：
``` ruby
coffee:
  square = (x) -> x * x

markdown:
  #Header
    Hello from #{"Markdown!"}
    Second Line!

css:
  .my_class {
    font: 12px/30px;
  }
```

### 编辑器
Textmate与Sublime的Bundle在[这里](https://github.com/slim-template/ruby-slim.tmbundle)
  
Emacs的slim-mode在这里[这里](https://github.com/slim-template/emacs-slim)
  
Vim的插件在[这里](https://github.com/slim-template/vim-slim)

### 转换erb或haml到slim
erb到slim的转换器在[这里](https://github.com/slim-template/html2slim)
  
haml到slim的转换器在[这里](https://github.com/slim-template/haml2slim)

### 总结
这篇文章浅浅介绍了Slim的安装和集成，语法和开发工具。没有介绍Slim的自定义缩写，Slim中引用Slim作为片段，支持的引擎，拥有的插件，和配置Slim。更深入的介绍在[官方的文档](http://rdoc.info/gems/slim/frames)，这里的大部分内容来自官方的文档。
  
由于octopress的代码嵌入不支持slim，（因为它是小众的追求极致的新模版语言），代码高亮的不是很好。我可能会在以后更新中修复这个问题。
  
对于Slim给人的第一印象，[官方网站](http://slim-lang.com)做得很精美。
  
欢迎评论和交流。

