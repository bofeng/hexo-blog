---
title: 'jQuery Tips'
date: 2015-11-23 23:15:48
tags: [jQuery,js]
published: true
hideInList: false
feature: 
---
Tips:

1, User variable to cache $element, avoid using $ to get element several times.

```javascript
$element = $('#element');
h = $element.height();
```

2, 命名方式上，在jQuery对象前加$前缀

3, 为了一致性，统一使用.on来绑定事件

```javascript
$ele.click(function() {});
// change to
$ele.on("click", function() {});
```

4, 优化选择符，例如#id是唯一的，没必要加额外的选择符

```javascript
// 不必要
$("div#myDiv")
```

5, 如果打算对dom做很多操作，可以先分离元素再添加：

```javascript
$ele = $container.first().detatch();
// a lot operation on this $ele
// then append it back
$container.append($ele);
```

6, “回到顶部” 按钮的动画：

```javascript
$('a.top').click(function (e) {
    e.preventDefault();
    $(document.body).animate({scrollTop: 0}, 800);
});
```

7, 自动修复损坏图片：

```javascript
$('img').on('error', function () {
    $(this).prop('src', 'img/broken.png');
});
```

8, 增加disabled和enabled的函数：

```javascript
$.fn.disabled = function () {
    return this.prop("disabled", true);
}
$.fn.enabled = function () {
    return this.prop("disabled", false);
}
// To use it:
$btn.disabled();
$btn.enabled();
```

9, 有时候不想让用户drag element的时候选中其中的文本：

```javascript
$.fn.disableSelection = function() {
    return this.attr('unselectable', 'on').css('user-select', 'none').on('selectstart', false);
}
// To use it:
$txt.disableSelection();
```

10, 通过使用contains选择器实现本页面文本搜索：

```javascript
var search = $('#search').val();
$('div:not(:contains("' + search + '"))').hide();
```

11, Ajax调用错误的处理
 当某次 Ajax 调用返回 404 或 500 错误，就会执行错误处理。但如果没有定义该处理，其他 jQuery 代码或许会停止工作。可以通过下面这段代码定义一个全局 Ajax 错误处理：

```javascript
$(document).ajaxError(function (e, xhr, settings, error) {
    console.log(error);
});
```
