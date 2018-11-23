应用场景：在前端开发的时候，经常会出现让用户上传头像的功能需求，除非是专业的美图工具可以直接人脸识别出用户上传的图片，一般用户在上传了图片后都可以对所上传的图片进行拖拽和缩放，将想要展示的部分拖放到规定的区域内，如微信上传头像和QQ上传头像。这次在所做活动中，有一部分的功能就是上传头像，即用户在上传图片后可拖拽和缩放图片，然后将图片裁切到特定区域，形成头像。
页面首页地址：/activity/activity-201801-newyear-main/index
### 实现原理
* 监听用户手指触摸事件，并做出相应判断，如果是单指滑动操作即为拖拽，如果是双指操作，则要判断是否为缩放手势；
* 用户对图像的拖拽和缩放操作其实是在canvas中对图像的不断重绘；
* 缩放的原理：取用户两指触点的坐标值，缩放的中心点即为两点的中点。
在缩放过程，绘制图像要根据图像的左上顶点绘制，牵一发而动全身。
![缩放原理图](https://img.aixuedai.com/null/2018227/19571547/20180227195715_305x380.png?height=380&width=305)
这里，由数学关系得出：
x-m = scale*(x0-m)
y-n = scale*(y0-n)
其中(m,n)为缩放的中点坐标，(x0,y0)为canvas中图像左上顶点原坐标，(x,y)为缩放后图像左顶点坐标，scale为缩放的比例，即为用户两指的距离缩放比例。
* 在用户对图像的缩放和拖动过程中，画布中不动的人脸轮廓部分可采用离屏canvas存储以提高性能。
* 对图像拖动和缩放操作后，记录下图像的左上顶点位置和图像尺寸参数，然后根据这些参数对图像在特定区域内进行裁切。
* 用canvas画不规则轮廓，可根据轮廓的svg路径在此在线工具生成<https://github.com/samsha/svg2canvas>
### 知识点梳理

* #### canvas重绘
canvas重绘，首先要擦除上一帧的canvas画布，然后再重新绘制。

* #### canvas清除画布的几种方法
较常用的使用clearRect方法：
```
let c = document.getElementById('canvasId')
let cxt=c.getContext("2d")
ctx.clearRect(0, 0, canvas.width, canvas.height)
```
由于canvas每当高度或宽度被重设时，画布内容就会被清空，因此可以用以下方法清空:
```
let c = document.getElementById('canvasId')
c.height=c.height
```

* #### 离屏canvas
离屏canvas又称为 缓冲canvas，幕后canvas。
应用场景：离屏canvas经常用来存放临时性的图像信息。在使用canvas制作动画的过程中，可以使用离屏canvas来存储不常变动的背景等，以此来提高动画的性能。
使用离屏canvas，通常要遵循下面四个步骤：
1. 创建用作离屏canvas的元素
2. 设置离屏canvas的宽度与高度
3. 在离屏canvas之中进行绘制
4. 将离屏canvas的全部或一部分内容复制到正在显示的canvas之中
具体代码见下：
```
let canvas = document.getElementById('canvas'),
    context = canvas.getContext('2d'),
    offsetscreenCanvas = document.createElement('canvas')

let offsetscreenContext = offsetscreenCanvas.getContext('2d')

offsetscreenCanvas.width = canvas.width
offsetscreenCanvas.height = canvas.height

offsetscreenContext.drawImage(anImage, 0, 0)

context.drawImage(offsetscreenCanvas, 0, 0, offsetscreenCanvas.width, offsetscreenCanvas.height)

```

#### drawImage方法
![drawImage图](http://hi.csdn.net/attachment/201108/4/0_1312439694SFqd.gif)
drawImage(image, dx, dy)
drawImage(image, dx, dy, dw, dh)
drawImage(image, sx, sy, sw, sh, dx, dy, dw, dh)
其中dx,dy 为在canvas中的左上顶点位置坐标，dw,dh为image在canvas中即将绘制区域（相对dx和dy坐标的偏移量）的宽度和高度值;sx,sy 为image所要绘制的起始位置，sw,sh是image所要绘制区域（相对image的sx和sy坐标的偏移量）的宽度和高度值。
值得注意的是，第一个参数image可以用HTMLImageElement，HTMLCanvasElement或者HTMLVideoElement作为参数

#### canvas 裁切
canvas使用clip()方法进行裁切，具体使用可参考<https://www.w3cplus.com/canvas/
clip.html>
在默认情况下，剪辑区域的大小与Canvas画布大小一致。调用Canvas绘图环境对象的clip()方法可以显式的设定剪辑区域。一旦设置好剪辑区域，那么你在Canvas之中绘制的所有内容都将局限在该区域内。
示例：
```
var canvas = document.getElementById('canvas')
var ctx = canvas.getContext('2d')
// Create clipping region
ctx.arc(100, 100, 75, 0, Math.PI * 2, false)
ctx.clip()
ctx.fillRect(0, 0, 100, 100)
```

#### canvas 中状态的保存和恢复
save 和 restore 方法是用来保存和恢复 canvas 状态的，都没有参数。

* Canvas 的状态就是当前画面应用的所有样式和变形的一个快照。一个画布的图形状态包含了 [CanvasRenderingContext2D](http://www.w3school.com.cn/jsref/dom_obj_canvasrenderingcontext2d.asp)对象的所有属性（除了只读的画布属性以外）。它还包含了一个变换矩阵，该矩阵是调用 rotate()、scale() 和 translate() 的结果。另外，它包含了剪切路径，该路径通过 clip() 方法指定。可是要注意，当前路径和当前位置并非图形状态的一部分，并且不会由这个方法保存。

* 使用save()方法其实是将canvas的绘制状态存入绘图状态栈中，使用restore()方法弹出绘图状态栈顶端的状态，并且根据这些值来设置当前绘图状态。
* 应用场景：在对canvas坐标系进行平移，缩放，旋转操作，或者是做裁切操作时候一般都要先保存之前的绘图状态，以免会对后面的绘图造成影响。

### 开发遇到的问题
在重绘图像的时候，要搞清楚图像的左上顶点在canvas中的坐标和在屏幕中坐标的不同。在canvas中绘制图像的时候，使用的是canvas中的坐标，而去处理手势操作时，使用的是在屏幕中的坐标。因此，在对图像拖拽和缩放过程中，有时候需要先把屏幕中的坐标转化为canvas中的坐标。代码如下所示：
```
windowToCanvas(x, y) {
    let bbox = this.canvasDom.getBoundingClientRect()
    return {
        x: x - bbox.left * (this.canvasWidth / bbox.width),
        y: y - bbox.top * (this.canvasHeight / bbox.height)
    }
}
```
说明：

* getBoundingClientRect()用于获取某个html元素相对于视窗的位置集合，执行 object.getBoundingClientRect()会得到元素的top、right、bottom、left、width、height属性，这些属性以一个对象的方式返回。具体可参考<https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect>
* canvas元素实际上有两套尺寸，一个是元素本身的大小，还有一个是元素绘图表面（drawing surface）的大小。以内联方式设置width和height属性，实际上是同时设置了元素本身的大小与元素绘图表面的大小。然而，如果通过CSS来设定canvas元素的大小，那么只会改变元素本身的大小，而不会影响到绘图表面。如果css设定的canvas元素大小域其绘图表面大小不同，浏览器就会对绘图表面进行缩放，使其符合元素的大小，类似于设定了图片的宽与高，但是与图片本身尺寸不同，那么，图片会被拉伸一样。
值得注意的是，在canvas元素大小与绘图表面大小不符合时，以上仍然适用。

### 存在的不足
由于开发时间比较紧俏，有一些不足之处可以优化。

* 对于手势操作的判断不够全面，如多指手势操作没做处理，双指操作只做了缩放处理，没做同向判断处理（如果是同向的话，应该作为拖动来处理）
* 在缩放的过程中，没对超出canvas边界做处理