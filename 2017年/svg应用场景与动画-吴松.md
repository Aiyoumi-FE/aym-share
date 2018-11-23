## svg应用场景与动画

### SVG简介
SVG（Scalable Vector Graphics）是可缩放矢量图形的缩写，基于可扩展标记语言XML来描述二维矢量图形的一种图形格式，由W3C制定，是一个开放标准。

![](http://nullundefined.coding.me/review/pages/20170406/img/image.png)

### svg基础

####svg基本图形
`rect(矩形)` `circle(圆)` `ellipse(椭圆)` `line(直线)` `polyline(折线)` `polygon(多边形)` `path(路径)`

####svg文本
`text(文本)` `textPath(文本路径)`

>以上svg元素这里不多说，不熟悉自己查看手册: [http://know.webhek.com/svg/svg-home.html](http://know.webhek.com/svg/svg-home.html)

###svg 使用方式与场景
**1,直接插入html文档**

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title></title>
</head>

<body>
    <svg xmlns="http://www.w3.org/2000/svg" width="300" height="300">
        <circle cx="150" cy="150" r="100" fill="#000" stroke="#e00000" stroke-width="2" />
    </svg>
</body>

</html>

```

**2,当普通图片使用**

```html
直接用<img>引入 :<img src="guaiguai.svg" alt="">
```
**3,当背景图使用**

``` css
background-image: url(img/guaiguai.svg);
```
**4,svg转base64 或 utf-8**

转化后的图片可以直接放到css中,跟其他png gif 图一样

![](http://nullundefined.coding.me/review/pages/20170406/img/demo-img.png)

**5,制作动画**

svg动画实现方式:

- svg动画元素(不推荐)   
`<animate> `	`<animateMotion>`	`<animateTransform>`	`<mpath>`

 想了解的请前往: [http://www.webhek.com/post/svg-animate-element.html](http://www.webhek.com/post/svg-animate-element.html)
 
 例子: [http://colorfonts.langustefonts.com/imgs/unicorn3.svg](http://colorfonts.langustefonts.com/imgs/unicorn3.svg)

- svg + css3

- svg + javascript


**6,图形图表**

目前主流的图表库`D3.js(svg)` `Echarts(canvas)` `highcharts(svg)`

**7,地图**

---

###svg动画案例
- loading菊花

<img src="http://nullundefined.coding.me/review/pages/20170406/img/loading.gif" width="200"/>

 demo前往:[http://nullundefined.coding.me/review/pages/20170406/svg-demo.html](http://nullundefined.coding.me/review/pages/20170406/svg-demo.html)

- 线条动画

 效果图如下:

<img src="http://nullundefined.coding.me/review/pages/20170406/img/line.gif" width="200">

<img src="http://nullundefined.coding.me/review/pages/20170406/img/heart.gif" width="200">

svg绘制线条动画有两个核心属性:`stroke-dasharray` `stroke-dashoffset`

- stroke-dasharray : 值为一组数字或百分比`"5, 10, 15"`  `"2%, 10%"` 第一个数代表实线部分，第二个数代表间隔部分...依次交替循环

```html
<style>
    line {
        stroke: #000;
        stroke-width: 2;
    }
</style>
<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200">
	<line stroke-dasharray="5,10" x1="0" y1="50" x2="200" y2="50" />
	<line stroke-dasharray="5,30" x1="0" y1="70" x2="200" y2="70" />
	<line stroke-dasharray="20,30" x1="0" y1="90" x2="200" y2="90" />
	<line stroke-dasharray="50%,50%" x1="0" y1="110" x2="200" y2="110" />
	<line stroke-dasharray="20%,50%" x1="0" y1="130" x2="200" y2="130" />
</svg>
```
<img src="http://nullundefined.coding.me/review/pages/20170406/img/line-array.png" width="200" style="border:1px solid#eee;"/>

- stroke-dashoffset: 线的偏移, 值为一个数字或百分比

弄明白这两个属性后，然后我们可以用css3`animation` 来动态改变`stroke-dasharray`或`stroke-dashoffset`, 达到线条动画的效果,代码如下:

```html
<style>
#line {
    animation: linemove 2s linear infinite;
    stroke-dasharray: 219px 219px;
}
  /* document.getElementById('line').getTotalLength() 获取路径长度 */
    
@keyframes linemove {
    0% {
        stroke-dashoffset: 219px;
    }
    100% {
        stroke-dashoffset: 0;
    }
}
</style>

<svg xmlns="http://www.w3.org/2000/svg" width="300" height="300" viewBox="0 0 100 100">
        <path id="line" d="M52.575 13.303a5.56 5.56 0 0 0-1.67-.515 6.295 6.295 0 0 0-.905-.084H38.003c-3.095 0-5.629 2.561-5.629 5.657s2.533 5.629 5.629 5.629h5.063L32.594 79.66c-1.913 8.1 9.045 10.751 11.356 3.042.869-4.424 3.737-5.769 6.049-5.766 2.312-.003 5.179 1.342 6.049 5.766 2.311 7.709 13.269 5.059 11.356-3.042l-11.78-62.623c-.533-1.996-1.763-3.131-3.049-3.734z" stroke="#000" fill="none"></path>
</svg>
```

效果请参见demo:[http://nullundefined.coding.me/review/pages/20170406/svg-animate.html](http://nullundefined.coding.me/review/pages/20170406/svg-animate.html)
[http://nullundefined.coding.me/review/pages/20170406/svg-heart.html](http://nullundefined.coding.me/review/pages/20170406/svg-heart.html)

---
### svg优化
**svg代码优化**

svgo:一款svg压缩工具,默认的 SVG 包含了许多可删除的不必要的信息，删除这些东西不会影响图像本身。下载安装前往: [https://github.com/svg/svgo](https://github.com/svg/svgo)

**svg sprite**


>**参考资料:**
>
>[http://know.webhek.com/svg/svg-home.html](http://know.webhek.com/svg/svg-home.html) [svg基础]
>
>[https://developer.mozilla.org/en-US/docs/Web/SVG](https://developer.mozilla.org/en-US/docs/Web/SVG)
>
>[http://www.alloyteam.com/2017/02/the-beauty-of-the-lines-break-lines-svg-animation/](http://www.alloyteam.com/2017/02/the-beauty-of-the-lines-break-lines-svg-animation/)[ svg线条动画]
>
>**svg相关工具:**
>
>[http://www.oschina.net/translate/20-useful-svg-tools-for-better-graphics](http://www.oschina.net/translate/20-useful-svg-tools-for-better-graphics)
>
>svg在线编辑器:[http://editor.method.ac/](http://editor.method.ac/)
>
>svg图标资料: [https://zhuanlan.zhihu.com/p/20753791](https://zhuanlan.zhihu.com/p/20753791)
