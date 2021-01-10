---
title: 修改主题--Yelee
date: 2020-02-29 09:18:21
tags: [Yelee, Hexo]
categories: 博客之路
---

> 对Hexo主题Yelee进行的一些修改。修复一些显示bug，添加新功能，对部分功能进行优化。

<!--more-->

### 移动端菜单栏显示问题
**目标文件** /source/css/_partial/main.styl。第二行，去掉container设置中`position: relative`前的注释符。此时，移动端菜单栏显示正常，但是在pc和移动端都会会出现右栏不能铺满屏幕的问题。继续更改css文件可以解决。

**目标文件** /source/css/style.styl。找到下面的代码段，在`html`和`body`中添加尺寸设置，修改为：
```git
html
  font-size base-font-size
+ height: 100%;
+ min-height: 100%;
+ min-width: 300px;

body
+ height: 100%;
+ min-height: 100%;
  font-family: font-sans, font-chs, sans-serif;
  background: #fff;
  text-rendering: optimizeLegibility;
  -moz-osx-font-smoothing: grayscale;
  -moz-font-feature-settings: "liga" on;
  color: #333
  -webkit-overflow-scrolling: touch
```

### 代码块前后留白问题
代码块中空行消失，导致显示出错，在issue上已经有人解决了。**目标文件：** source/css/_partial/highlight.sytl。修改为：
```git
  pre, code
    font-fanmily: font-mono, monospace, font-chs
    font-size: 1em
+ .line:after
+   content: ''
+   display: inline-block;
  pre
    @extend $code-block
    code
```


### 取消用新标签页面打开搜索结果
**目标文件：** themes/yelee/source/js/search.js。找到下面代码段
```java
// show search results
if (isMatch) {
    str += "<li><a href='"+ data_url +"' class='search-result-title' target='_blank'>"+ "> " + data_title +"</a>";
```
删除`target='_blank'`字段


### 取消迷你文章在新窗口打开
**目标文件：** themes/yelee/layout/_partial/open-in-new-tab.ejs。找到
```java
<% if (theme.open_in_new.mini_archives) { %> miniArchives: "a.post-list-link", <% } %>
```
因为在_config.yml中设置mini_archives好像不起作用，所以将`theme.open_in_new.mini_archives`直接设为false即可。


### 添加文章字数统计
这个功能需要用到插件：`hexo-wordcount`，在Hexo项目根目录下使用命令`npm i --save hexo-wordount`进行安装。下面的配置是信息显示位置和样式，仅供参考。

**目标文件：** themes/yelee/layout/_partial/post/tag.ejs。做如下修改：
```git
<% if (post.tags && post.tags.length){ %>
-    <div class="article-tag tagcloud">
+    <div class="article-tag tagcloud" style="display: flex; flex-wrap: wrap">
        <%-
          list_tags(post.tags, {
            show_count: false,
            class: 'article-tag'
          })
        %>
+       <span class="post-count">总字数<%= wordcount(post.content) %></span>
+       <span class="post-count">预计阅读<%= min2read(post.content) %>分钟</span>
    </div>
<% } %>
```
添加样式：
**目标文件** themes/yelee/source/css/_partial/tagcloud.styl。找到
```css-extras
.article-tag::before,
.article-category::before
    float left
    color #999
    font base-font-size FontAwesome
    margin-right 5px
    margin-top (1/3)rem

.article-tag::before
    content "\f02b"
    margin-left 1em

.article-category::before
    content "\f02d"
```
在`.article-category::before`前插入以下代码
```css-extras
.article-tag
    .article-tag-list
        display flex
        flex-wrap wrap
    span
        cursor pointer
        line-height 29px
        font-size 13px
        color #aaa
        &:before
            content "\27A4"
            margin-left 1em<Paste>
```
其中，`content: "\27A4"`是unicode图标编码，根据自己需求替换。

[unicode图标](https://unicode-table.com/cn)

[Font Awesome icom](https://fontawesome.com/icons?from=io)


### 添加多合一打赏
实现原理见博文, [多合一收款二维码原理及实现(源码)](https://mkblog.cn/922/)。具体操作为，在yelee的配置文件
`_config.yml`中加上：
```yml
# 打赏
# 将 on 改为 false 去掉打赏
donate:
  on: true
  multipay: /img/multipay.png
```
multipay.png上的二维码指向链接/pages/donate，然后根据用户的userAgent转向不同服务。

**目标文件** themes/yelee/layout/_partial/left-col.ejs。找到
```java
<nav class="header-nav">
    <ul class="social">
        <% for (var i in theme.subnav){ %>
            <a class="fa <%= i %>" href="<%- url_for(theme.subnav[i]) %>" title="<%= i %>"></a>
        <%}%>
    </ul>
</nav>
```
在`</ul>`下添加
```java
<% if (theme.donate.on) { %>
    <ul class="social">
        <div style="position: absolute; top: 95%; left: 50%; margin-left: -30px;">
            <p style="display: block">
                <a
                    class="donateIcon"
                    href="javascript:void(0)"
                    onmouseout="
                        var qr = document.getElementById('donate');
                        qr.style.display='none';
                    "
                    onmouseenter="
                        var qr = document.getElementById('donate');
                        qr.style.display='block';
                    ">
                    赏
                </a>
            </p>
        </div>
    </ul>
<% } %>
```
在`left-col.ejs`文件末尾添加：
```java
<% if (theme.donate.multipay) { %>
    <div id="donate">
        <img id="multipay" src="<%=theme.donate.multipay%>" width="250px" alt="<%=theme.author%> Multipay"/>
        <div class="triangle"></div>
    </div>
<% } %><Paste>
```

**目标文件：** themes/yelee/source/css/_partial/customise/social-icon.styl。在顶部找到
```css-extras
#header .header-nav .social
    margin-top 10px
    text-align center
    font-family Arial
    a
        width base-font-size + 21
        height @width
        border-radius 50%
        margin 0 2px 6px
        vertical-align middle
        font-size .66*@width
        line-height @width
        text-align center
        color white
        background #6f7170
        opacity i-opacity
        box-shadow 1px 2px 2px rgba(0,0,0, .1), 1px 1px 3px rgba(0,0,0, .3)
        &:hover
            opacity 1
            transform scale(1.1)
```
在`a`标签的样式后添加
```css-extras
.donateIcon
    display block
    width 56px
    margin auto
    height 56px
    font-size 20px
    color #fff
    border none
    background #4094c7
    border-radius 50%
    text-align center
    -webkit-box-shadow 0 2px 5px 0 rgba(0,0,0,0,16), 0 2px 10px 0 rgba(0,0,0,0,12)
    box-shadow 0 2px 5px 0 rgba(0,0,0,0,16), 0 2px 10px 0 rgba(0,0,0,0,12)
    -webkit-transition 0.4s ease-in-out
    -moz-transition 0.4s ease-in-out
    -ms-transition 0.4s ease-in-out
    transition 0.4s ease-in-out
```
**目标文件：** themes/yelee/source/css/_partial/main.styl。找到`intrude-less`并加上donate标签的样式。
```git
.intrude-less {
    width: 76%;
    text-align: center;
    margin: 112px auto 0;
+   #donate {
+       display: none;
+       position: fixed;
+       top: 280px;
+       left: 25px;
+       img {
+           border-radius: 5px;
+       }
+       .triangle {
+           height: 0;
+           width: 0;
+           margin: -5px 0 0 85px;
+           border-right: 45px solid transparent;
+           border-left: 45px solid transparent;
+           border-top: 30px solid #5b91ee;
+       }
+   }
}
```


### 修改列表显示的bug
markdown文件中无序列表显示有误，比如 “- [列表名]”, 如果列表名前两个字符含 ‘x’, 则列表前缀消失。排查之后发现是 main.js 里对无序列表进行了判断，若出现 “- [ ]” 或者 “- [x]”，则解析成复选框的样式，需要修改判断条件。

**目标文件：** themes/yelee/source/js/main.js
```git
// Task lists in markdown
$('ul > li').each(function() {
    var taskList = {
-       field: this.textContent.substring(0, 2),
+       field: this.textContent.substring(0, 3),
        check: function(str) {
-           var re = new RegExp(str);
-           return this.field.match(re);
+           return this.field === str;
        }
    }
    ...
}
```


### 添加谷歌站长并提交站点地图
验证网站：[Search Console](https://www.google.com/webmasters/tools/)。然后安装sitemap生成器，`npm install hexo-generator-sitemap --save`，在站点配置文件`_config.yml`中加上
```yml
### sitemap
sitemap:
  path: sitemap.xml
```
再部署一遍后，博客目录下会多出一个`sitemap.xml`文件，到谷歌站长里添加地图即可。

值得注意的是，搜索引擎会抓取路径在三级以内的地址，如果路径太多可能抓取不到，但 hexo 文章的路径默认会加上日期，比如`http://wangriyu.wang/2017/08/24/Hexo/`。在站点配置文件中修改一下permalink（默认是:year/:month/:day/:title/），这样就不会在加上日期了：
```yml
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://wangriyu.wang
root: /
permalink: :title.html
permalink_defaults:
```


### 使用七牛云图床
注册[七牛云](https://sso.qiniu.com/)账号，七牛云可以用来生成外链来代替站内的图片资源。详情略过。


### 添加隐藏侧边栏的按钮
**目标文件：** themes/yelee/layout/layout.ejs
```git
    ...
+   <div class="hide-left-col" title="隐藏侧栏">
+     <i class="fa fa-angle-double-left"></i>
+   </div>
    <div class="mid-col">
      <%- partial('_partial/mobile-nav', null, {cache: !config.relative_link}) %>
      <div class="body-wrap"><%- body %></div>
      <%- partial('_partial/footer') %>
    </div>
+   <script type="application/javascript">
+     var hide = false, leftWidth = <%- theme.left_col_width %>;
+     function hideLeftCol() {
+       if (hide) {
+         $(".left-col").width(leftWidth);
+         $(".left-col .intrude-less").css('display', '');
+         $("#tocButton").css('display', '');
+         ($('#switch-btn').css('display') === 'block' && $('#switch-area').css('display') === 'block') || $('#toc').slideDown(320);
+         $(".hide-left-col").css("left", leftWidth).html('<i class="fa fa-angle-double-left"></i>');
+         $(".mid-col").css("left", leftWidth)
+         $("#post-nav-button").css("left", leftWidth)
+         $("#post-nav-button > a:nth-child(2)").css("display", "block")
+         hide = false
+       } else {
+         $(".left-col").width(0);
+         $(".left-col .intrude-less").css('display', 'none');
+         $("#toc").css('display', 'none');
+         $("#tocButton").css('display', 'none');
+         $(".hide-left-col").css("left", 0).html('<i class="fa fa-angle-double-right"></i>');
+         $(".mid-col").css("left", 0)
+         $("#post-nav-button").css("left", 0)
+         $("#post-nav-button > a:nth-child(2)").css("display", "none")
+         if ($(".post-list").is(":visible")) {
+           $("#post-nav-button .fa-bars,#post-nav-button .fa-times").toggle();
+           $(".post-list").toggle();
+         }
+         hide = true
+       }
+     }
+     $(".hide-left-col").click(function() {
+       hideLeftCol()
+     });
+   </script>
    <%- partial('_partial/after-footer') %>
  </div>
```
**目标文件：** themes/yelee/source/css/_partial/main.styl。
```git
.left-col {
  ...
}
+.hide-left-col {
+   z-index: 999;
+   cursor: pointer;
+   transition:all .2s ease-in;
+   position: fixed;
+   top: 0;
+   left: left-col-width;
+   text-align: center;
+   line-height: 30px;
+   display: block;
+   width: 30px;
+   height: 30px;
+   font-size: 28px;
+   background: none;
+   .fa {
+       color: rgb(255, 255, 255, .5);
+   }
+   &:hover {
+       background: rgba(147, 181, 224, .3) !important;
+       .fa {
+           color: white;
+       }
+   }
+}
.mid-col {
  ...
}

...

@media screen and (max-width:768px) {
+   .hide-left-col {
+       display: none;
+   }
    @import "_partial/mobile"
}

...

#post-nav-button {
    left: left-col-width;
    top: 61.8%;
+   transition: left .2s ease-in;
    a {
        border-bottom-color: transparent;
        background: none;
        cursor: pointer;
        .fa-times {
            display: none;
        }
    }
}
```
**目标文件：** themes/yelee/source/css/_partial/mobile.styl。
```git
.left-col {
-   display: none;
+   display: none !important;
}
.mid-col {
-   left: 0;
+   left: 0 !important;
}
```


### 使用prism代码高亮
安装插件` npm i -S hexo-prism-plugin`。将根目录`_config.yml`中的highlight部分均设为false，并添加prism配置项：
```git
highlight:
- enable: true
+ enable: false
  line_number: false
  auto_detect: false
  tab_replace:

# prism
# https://prismjs.com/#languages-list
+ prism_plugin:
+   mode: 'preprocess'   # realtime/preprocess
+   theme: 'a11y-dark'   # https://github.com/PrismJS/prism-themes#available-themes
+   line_number: true    # default false
+   custom_css:          # optional
```
将主题的`_config.yml`的highlight_style删掉。
```git
-highlight_style:
-  on: false
-  inline_code: 3  # Value: 0 - 9 可选
-  code_block: 4  # Value: 0 - 4
-  # Set inline_code to style highlight text
-  # Chose a highlight theme for code block
-  # 通过 inline_code 切换内置文本高亮样式
-  # 通过 code_block 切换内置代码高亮配色主题
```
删除 themes/yelee/source/css/_partial/customise/ 下的 code-block.styl 和 inline-code.styl。删除 themes/yelee/source/css/_partial/highlight.styl

修改mobile.styl:
```git
    .article-entry{
        padding-left: 0;
        padding-right: 0;
-       .highlight {
-           padding .35em .6em
-       }
    }
```
修改themes/yelee/source/css/style.styl:
```git
@import "_partial/archive"
@import "_partial/article"
@import "_partial/archive"
- @import "_partial/highlight"
@import "_partial/footer"

if share
@ -117,4 +116,14 @@ if search
@import "_partial/customise/color-scheme"

if sidebar
  @import "_partial/sidebar"
  @import "_partial/sidebar"

+ .article-entry code
+   color gray
+   padding .05em .3em
+   border-radius 3px
+   box-shadow 1px 1px 1px rgba(0,0,0, .08)
+   background rgba(255, 250, 165, .7)

+ .article-entry pre code
+   padding 0
```
注意使用此插件时所有代码块都需要指定语言，否则无法加载高亮效果。[语言列表](https://prismjs.com/#languages-list)


### 升级fancybox图片浏览插件
修改主题的`_config.yml`中的CDN：
```git
-  fancybox_js: //cdn.bootcss.com/fancybox/2.1.5/jquery.fancybox.min.js
-  fancybox_css: //cdn.bootcss.com/fancybox/2.1.5/jquery.fancybox.min.css
+  fancybox_js: //cdn.bootcss.com/fancybox/3.3.5/jquery.fancybox.min.js
+  fancybox_css: //cdn.bootcss.com/fancybox/3.3.5/jquery.fancybox.min.css
```
修改themes/yelee/source/js/main.js:
```git
// fancyBox
if(!!yiliaConfig.fancybox){
    require([yiliaConfig.fancybox_js], function(pc){
        var isFancy = $(".isFancy");
        if(isFancy.length != 0){
            var imgArr = $(".article-inner img");
            for(var i=0,len=imgArr.length;i<len;i++){
                var src = imgArr.eq(i).attr("src");
                var title = imgArr.eq(i).attr("alt");
                if(typeof(title) == "undefined"){
                    var title = imgArr.eq(i).attr("title");
                }
                var width = imgArr.eq(i).attr("width");
                var height = imgArr.eq(i).attr("height");
-               imgArr.eq(i).replaceWith("<a href='"+src+"' title='"+title+"' rel='fancy-group' class='fancy-ctn fancybox'><img src='"+src+"' width="+width+" height="+height+" title='"+title+"' alt='"+title+"'></a>");
+               imgArr.eq(i).replaceWith("<a class='fancy-ctn' href='"+src+"' title='"+title+"' data-fancybox='images' data-caption='"+src.substring(src.lastIndexOf("/")+1)+"'><img src='"+src+"' width="+width+" height="+height+" title='"+title+"' alt='"+title+"'></a>");
            }
-           $(".article-inner .fancy-ctn").fancybox({ type: "image" });
+           $(".article-inner .fancy-ctn").fancybox({
+             loop: true,
+             touch: false,
+             toolbar: true,
+             wheel: false,
+             buttons: [
+               "fullScreen",
+               "thumbs",
+               "close"
+             ],
+           });
        }
    })
}
```
修改themes/yelee/source/css/style.styl，在最后面插入：
```js
.fancybox-image
  background #ffffff
```


## 相关链接
---
1. [Yelee主题使用说明, **By** MOxFIVE](http://moxfive.coding.me/yelee/)
2. [使用Hexo主题Yelee, **By** 鱼の乐](https://blog.wangriyu.wang/2017/08-Hexo.html)



[使用Hexo主题Yelee, **By** 鱼の乐]: https://blog.wangriyu.wang/2017/08-Hexo.html
