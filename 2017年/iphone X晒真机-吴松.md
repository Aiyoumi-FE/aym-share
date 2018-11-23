### 开发前的思考与难点
背景图有高斯模糊，手机中显示的图没有。背景图和手机显示图有个显示比例差,所以在绘制的时候他们是两层。第一层背景高斯模糊层，要解决高斯模糊;第二层手机显示层，绘制这个层，需要绘制这个不规则的裁切区域。 图片的移动与缩放，这两层要实现联动，达到所见即所得的效果。

[活动快速入口](https://m.aiyoumi.com/activity/activity-201709-iphone8_show-main/index)

#### 涉及的主要技术
* 离屏canvas
* svg路径的应用
* getImageData   putImageData
* 画布缩放与平移

#### 难点
* 高斯模糊
* 背景拖动缩放， 与手机中的图联动
* 手机区域的图裁切
* 背景图与手机显示图的 边界计算
* 屏幕适配

#### 遇到问题与解决
* 高斯模糊方法中在使用getImageData时报错: getImageData 返回一个ImageData对象， 包含data width height 三个只读属性，直接给imageData.data 赋值会报错,但是imageData.data[n]是可以写的
* getImageData性能问题, 当touchmove时候，一直调用getImageData会卡顿。可以先将高斯模糊图保存下来，这样就不会频繁调用getImageData
* 平移和缩放，都会影响你的绘图,在没有ctx.restore()之前， 你得计算你的绘制坐标
* 手机显示区域是一个裁切区域，需要绘制这个区域的路径，手动绘制太复杂；想到能不能将svg转换成canvas绘制代码[https://github.com/samsha/svg2canvas](https://github.com/samsha/svg2canvas)

#### 不足之处
* 动态二维码生成使用的qrcode库,  生成二维码在不同机型密度会有些差异。
* 在对图片进行移动与缩放时，不是原始图片。而已已经改变过尺寸的。 所以当你缩放时，清晰度会比较差。
* 在拖动与缩放时， 没有加缓动动画。所以当你进行拖动与缩放时，会显得干涩
