---
layout : life
title: bootstrap docs
category : server
duoshuo: true
date : 2018-02-20
---

# Bootstrap

1. Media Query
```css
@media (min-width:992px) {
    /* CSS Style*/
}
```

2. 针对移动网页优化
* <meta name="viewport" content="width=device-width,initial-scale=1,shrink-to-fit=no">

3. bootstrap grid
* pull-sm-5, push-sm-7, 在小屏幕下，左移5个，右移7个grid。
* flex-sm-first, flex-sm-last,在小屏幕下，成为第一个元素，成为最后一个元素。
* class="row align-items-center", 在`竖直`方向`居中`。
* class="row justify-content-center", 在内部如果有class="col-auto", 基于内容自动调整占用列数,用于`水平居中`。
* class="col-sm-4 offset-sm-1",右移1个格子。

4. classes
* <header class="jumbotron"> 适用超大屏幕
* col-sm 不指定数字就是剩余所有 

5. Something
* <ul class="list-unstyled">
* text-align: center
* color: ** 是字体的颜色

6. href
* tel:+86xxx 调用外部打电话软件
* mailto:xxx 调用发邮件

7. label 和 input
* <label for="firstname"
* <input id="firstname"
* label的`for`与input的`id`对应，在点击label的文字时，相应的input会被激活。

8. closet tag
* <textarea></textarea> 需要闭合，否则页面无法显示。
* <input>无需闭合

9. media
* 嵌入一个内容
```go
<div class="embed-responsive embed-responsive-16by9">
    <iframe class="embed-responsive-item" src="https://xxx.com"></iframe>
    <!-- <embed> <video> <object>-->
</div>
```

10. margin
* margin top 0: `class="mt-0"`
* margin left 3: `class="ml-3"`

11. dispaly: flex
* webkit前缀: display: -webkit-flex
* flex-shrink.

12. class="hidden-xs-down"
在超小屏幕下隐藏