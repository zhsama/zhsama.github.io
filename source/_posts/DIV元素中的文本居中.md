```
---
title: DIV元素中的文本居中
date: 2022-02-24 01:44:24
updated: 2022-02-24 01:44:28
tags: ['css']
categories: css
description: 被垂直布局困扰的我，终于受不了每次布局都要查询的麻烦，总结了一下常用的div文本居中布局。
---
```

# DIV元素中的文本居中

在样式布局中，我们经常碰到需要将元素居中。

通过css实现元素的水平居中较为简单：对文本，只需要对其父级元素设置`text-align: center;`，而对div等块级元素，只需要设置其 left 和 right 的 `margin: auto;`。

要实现元素的垂直居中，很容易便会想到`vertical-align`属性，但是该属性只对拥有`valign`特性的元素才生效，例如表格元素中的`<td>`、`<th>`、`<caption>`等。而像`<div>`、`<span>`这样的元素是没有`valign`特性的，使用该属性对这些元素并不起作用。

因此我们需要通过别的方法去实现元素的垂直居中，我总结了几种了常用垂直居中方法在下面。

# 单行文本垂直居中

## 单行文本垂直居中

对于单行文本，我们只需要将文本行高(line-height)和所在区域高度(height)设为一致即可：

   ```html 
   <div class="div1">
   	这是单行文本垂直居中
   </div>
   <style>
   .div1{
   	width: 300px;
   	height: 100px;
   	margin: 50px auto;
   	border: 1px solid red;
   	line-height: 100px; /*设置line-height与父级元素的height相等*/
   	text-align: center; /*设置文本水平居中*/
   	overflow: hidden; /*防止内容超出容器或者产生自动换行*/
   }
   </style>
   ```

# 多行文本垂直居中

多行文本垂直居中分为两种情况：

1. 是父级元素高度不固定，随着内容变化
2. 父级元素高度固定

## 父级元素高度不固定

父级高度不固定的时，高度只能通过内部文本来撑开。这样，我们可以通过设置内填充（padding）的值来使文本看起来垂直居中，只需设置`padding-top`和`padding-bottom`的值相等。

代码如下：

```html
<div class="div1">
	这是多行文本垂直居中，
	这是多行文本垂直居中，
	这是多行文本垂直居中，
	这是多行文本垂直居中。
</div>
<style>
.div1{
	width: 300px;
	margin: 50px auto;
	border: 1px solid red;
	text-align: center; /*设置文本水平居中*/
    padding: 50px 20px;
}
</style>
```

## 父级元素高度固定

在本文一开始就提到了`vertical-align`属性只对拥有 valign 特性的元素才生效，所以可以结合`display: table;`，使得 div 元素模拟 table 的属性。

具体操作为：设置父级div为`display: table;`，添加一个子 div 用于显示文本内容，并给该子 div 元素设置`display: table-cell;`和`vertical-align: middle;`属性。

具体代码如下：

```html
<div id="outer">
    <div id="middle">
        这是固定高度多行文本垂直居中，
        这是固定高度多行文本垂直居中，
        这是固定高度多行文本垂直居中，
		这是固定高度多行文本垂直居中。
    </div>
</div>
<style>
#outer{
    width: 400px;
    height: 200px;
    margin: 50px auto;
    border: 1px solid red;
 	display: table;
}
#middle{ 
    display:table-cell; 
    vertical-align:middle;  
    text-align: center; /*设置文本水平居中*/  
    width:100%;   
}
</style>
```

# 子div垂直居中

## 1、根据子div具体大小设置偏移

如果子div固定大小，设定水平和垂直偏移父元素的50%，再根据实际长度将子元素向上和向左挪回一半大小

```html
<div class="outer">
	<div class="middle">
		子div(固定大小)垂直居中
	</div>          
</div>

<style>
.outer{
	background-color: #13CDF4;
    width: 300px;
    height: 200px;
    position: relative;
}
.middle{ 
	background-color: #E41627;
    width: 100px;
    height: 100px;
    margin: auto;
    position: absolute;
    left: 50%; 
    top: 50%;
    margin-left: -50px;
    margin-top: -50px;
}
</style>

```

## 2、translate

针对第一种方法中水平和垂直偏移父元素的50%后，不设置 margin 值，而是利用除css3中的 transform 属性设置 translate 的值，css代码部分改成如下：

```html
<style>
middle{ 
    background-color: #E41627;
    width: 100px;
    height: 100px;
    margin: auto;
    position: absolute;
    left: 50%; 
    top: 50%;
    transform: translateX(-50%) translateY(-50%);
	-webkit-transform: translateX(-50%) translateY(-50%);
}
</style>
```

这种方法需要注意transform是css3中的属性，使用时注意浏览器的兼容性，IE9之前的版本不支持。

## 3、absolute

利用绝对定位实现子 div 大小不固定垂直居中：

```html
<div class="outer">
	<div class="middle">
		绝对布局absolute垂直居中
	</div>          
</div>

<style>
.outer{
    background-color: #13CDF4;
    width: 300px;
    height: 200px;
	position: relative;
}
.middle{
    background-color: #E41627;
    width: 100px;   //子div大小可随意设置
    height: 100px;
    margin: auto;
    position: absolute;
	top: 0;left: 0;right: 0;bottom: 0;
}
</style>
```

该方法不兼容IE7、IE6

## 4、vertical-align

利用`vertical-align`属性实现子div大小不固定垂直居中：

```html
<div class="outer">
	<div class="middle">
    	利用vertical-align属性实现子div大小不固定垂直居中
	</div>          
</div>
<style>
.outer{
	background-color: #13CDF4;
    width: 300px;
    height: 200px;
    display: table-cell; 
    vertical-align: middle;
}
.middle{ 
	background-color: #E41627;
    width: 100px;
    height: 100px;
    margin: 0 auto;
}
</style>
```

这种方法是将div转变成table-cell显示，然后通过`vertical-align: middle;`再设置其子元素垂直居中，这种方法和上面设置父级元素高度固定时多行文本居中的方法一样，所以这种方法也不能兼容IE7、IE6。

## 5、利用display: flex

```html
<div class="outer">
    <div class="middle">
        利用display: flex实现子div大小不固定垂直居中
    </div>          
</div>

<style>
.outer{
	background-color: #13CDF4;
    width: 300px;
    height: 200px;
    display: flex;
    justify-content: center;/*实现水平居中*/
    align-items:center; /*实现垂直居中*/
}
.middle{ 
	background-color: #E41627;
    width: 100px;
    height: 100px;
}
</style>
```


这种方法只需要在父级div中加上这三句话就行，但是在IE中兼容性不好，IE9及以下IE浏览器版本都不支持。

# 总结

以上是笔者总结的一些常用到的垂直居中的布局方法，读者可以根据自己的需要选择合适的布局方式。
