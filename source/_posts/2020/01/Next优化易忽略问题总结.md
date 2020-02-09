---
title: Next优化易忽略问题总结
tags: 
  - Next
  - hexo主题
date: 2020-02-08 01:07:17
categories: 博客
---
这里记录了一些常见的Next优化过程中易忽略的问题，或者网上不好查找的解决办法。
<!--more-->
## 去掉"发表于"，添加"更新于"，并且去掉汉字只显示日期
1.1 打开`/app/blog/themes/next/languages`目录下的`zh-Hans.yml`（因为我用的是中文语言所以选择这个）
```
post:
  created: 创建于
  modified: " " # 表示更新时间  
  sticky: 置顶 
  posted: " " # 表示发表时间
  in: " " # 表示分类
  read_more: 阅读全文
  untitled: 未命名
  toc_empty: 此文章未包含目录
  visitors: 阅读次数
  wordcount: 字数统计
  min2read: 阅读时长
  # 以上三处用引号加空格代替即可去掉汉字现实。
```
1.2 打开`/app/blog/themes/next`下的主题配置文件
搜索`updated_at`，改其值false为true即可。
## 首页底部显示`<i class="fa fa-angle-right"></i>`错误
1.1 编辑`\themes\next\layout\_partials\pagination.swig`
```
{% if page.prev or page.next %}
  <nav class="pagination">
    {{
      paginator({
        prev_text: '<i class="fa fa-angle-left"></i>',
        next_text: '<i class="fa fa-angle-right"></i>',
        mid_size: 1
      })
    }}
  </nav>
{% endif %}
```
修改为
```
{%- if page.prev or page.next %}
  <nav class="pagination">
    {{
      paginator({
        prev_text: '<i class="fa fa-hand-o-left" aria-label="' + __('accessibility.prev_page') + '"></i>',
        next_text: '<i class="fa fa-hand-o-right" aria-label="' + __('accessibility.next_page') + '"></i>',
        mid_size : 1,
        escape   : false
      })
    }}
  </nav>
{%- endif %}
```
## 实现代码复制功能
方法一： 使用clipboard插件实现。
① 下载[cllipboard.min.js](https://raw.githubusercontent.com/zenorocha/clipboard.js/master/dist/clipboard.min.js)
或者使用（[备用下载地址](https://files.lucfzy.com/blog/clipboard.min.js)）
② 在`.\themes\next\source\js\src` 目录下创建`clipboard.min.js`并保存刚刚下载的文件内容
③在`.\themes\next\source\js\src`目录下创建`clipboard-use.js`，编辑如下内容
```
/*页面载入完成后，创建复制按钮*/
!function (e, t, a) { 
  /* code */
  var initCopyCode = function(){
    var copyHtml = '';
    copyHtml += '<button class="btn-copy" data-clipboard-snippet="">';
    copyHtml += '  <i class="fa fa-globe"></i><span>copy</span>';
    copyHtml += '</button>';
    $(".highlight .code pre").before(copyHtml);
    new ClipboardJS('.btn-copy', {
        target: function(trigger) {
            return trigger.nextElementSibling;
        }
    });
  }
  initCopyCode();
}(window, document);
```
④ 在`.\themes\next\source\css\_custom\custom.styl`样式文件中添加下面代码：
```
//代码块复制按钮
.highlight{
  //方便copy代码按钮（btn-copy）的定位
  position: relative;
}
.btn-copy {
    display: inline-block;
    cursor: pointer;
    background-color: #eee;
    background-image: linear-gradient(#fcfcfc,#eee);
    border: 1px solid #d5d5d5;
    border-radius: 3px;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    -webkit-appearance: none;
    font-size: 13px;
    font-weight: 700;
    line-height: 20px;
    color: #333;
    -webkit-transition: opacity .3s ease-in-out;
    -o-transition: opacity .3s ease-in-out;
    transition: opacity .3s ease-in-out;
    padding: 2px 6px;
    position: absolute;
    right: 5px;
    top: 5px;
    opacity: 0;
}
.btn-copy span {
    margin-left: 5px;
}
.highlight:hover .btn-copy{
  opacity: 1;
}
```
⑤ 引用文件，在`.\themes\next\layout\_layout.swig`文件中，添加引用（注：在 swig 末尾或 body 结束标签（</body>）之前添加）：
```
  <!-- 代码块复制功能 -->
  <script type="text/javascript" src="/js/src/clipboard.min.js"></script>  
  <script type="text/javascript" src="/js/src/clipboard-use.js"></script>
```
方法二：自定义函数实现代码复制
①在`themes/next/layout/_third-party/`目录下创建`copy-code.swig`，并编辑如下内容
```
  <style>
    .copy-btn {
      display: inline-block;
      padding: 6px 12px;
      font-size: 13px;
      font-weight: 700;
      line-height: 20px;
      color: #333;
      white-space: nowrap;
      vertical-align: middle;
      cursor: pointer;
      background-color: #eee;
      background-image: linear-gradient(#fcfcfc, #eee);
      border: 1px solid #d5d5d5;
      border-radius: 3px;
      user-select: none;
      outline: 0;
    }
 
    .highlight-wrap .copy-btn {
      transition: opacity .3s ease-in-out;
      opacity: 0;
      padding: 2px 6px;
      position: absolute;
      right: 4px;
      top: 8px;
    }
 
    .highlight-wrap:hover .copy-btn,
    .highlight-wrap .copy-btn:focus {
      opacity: 1
    }
 
    .highlight-wrap {
      position: relative;
    }
  </style>
 
  <script>
    $('.highlight').each(function (i, e) {
      var $wrap = $('<div>').addClass('highlight-wrap')
      $(e).after($wrap)
      $wrap.append($('<button>').addClass('copy-btn').append('复制').on('click', function (e) {
        var code = $(this).parent().find('.code').find('.line').map(function (i, e) {
          return $(e).text()
        }).toArray().join('\n')
        var ta = document.createElement('textarea')
        document.body.appendChild(ta)
        ta.style.position = 'absolute'
        ta.style.top = '0px'
        ta.style.left = '0px'
        ta.value = code
        ta.select()
        ta.focus()
        var result = document.execCommand('copy')
        document.body.removeChild(ta)
 
        if(result)$(this).text('复制成功')
        else $(this).text('复制失败')
 
        $(this).blur()
      })).on('mouseleave', function (e) {
        var $b = $(this).find('.copy-btn')
        setTimeout(function () {
          $b.text('复制')
        }, 300)
      }).append(e)
    })
  </script>
```
② 返回上一级目录编辑`_layout.swig`文件，搜索`commons.swig`，在这一行代码下面加入
```
{% include '_third-party/copy-code.swig' %}
```
即可








