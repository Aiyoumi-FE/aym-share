#### 说说开发前的感觉
> 刚开始拿到需求的时候，有好多技术难点我完全不知道，比如图片美化和透视(还好这个被砍了)，长按保存图片，怎么样讲照片绘制到合适的位置等等。当时只知道简单的绘制图片和文本，所以心里还是没底的。这个时候，只能一步一步来，硬上了。 
> 
> 活动链接:[https://m.aiyoumi.com/activity/activity-201707-iphone8-main/index/](https://m.aiyoumi.com/activity/activity-201707-iphone8-main/index/)

#### 开发中碰到的问题
> * 图片裁剪
> * 图片定位
> * canvas跨域
> * 手机端拍照上传，照片转向
> * 图片过大，无法上传
> * ios 和 安卓下input file控件, 选择拍照项，然后取消的表现差异
> * 安卓下 input file控件差异, 具体调用参见[capture](https://www.w3.org/TR/html-media-capture/)属性 
> * 长按识别图片中二维码问题(app 微信 QQ 以及其他浏览器表现各不相同)
> * vuex数据丢失问题(从页面1传到页面2, 页面2刷新时, 值丢失)

#### 做的相关优化
> * vue语法
> * 上传照片压缩处理
> * 上传照片中，增加loading
> * 合成照片, 增加“生成中...”提示
> * 增加对localStorage能否使用的判断(主要针对在无痕模式下浏览)

#### 解决问题
* 图片裁切： 用到canvas的[clip](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/clip)方法
* 图片定位: 图片宽高是不确定的，需要先判断宽高，然后对图片进行裁切

```javascript
if (photoImg.width <= photoImg.height) {
        pWidth = photoWidth
        pHeight = photoWidth
    } else {
        pWidth = photoHeight
        pHeight = photoHeight
        pX = (photoWidth - photoHeight) / 2
    }
context.drawImage(photoImg, pX, pY, pWidth, pHeight, -radius, -radius, radius * 2, radius * 2)
```

*  canvas跨域: 当使用跨域图片在cnavas上绘制图片的时候,就无法读取canvas上数据,相关方法将无法使用,例如:oBlob(), toDataURL() 或 getImageData() [参考资料](https://developer.mozilla.org/zh-CN/docs/Web/HTML/CORS_enabled_image),可使用image的crossOrigin属性

```javascript
let img = new Image()
    img.addEventListener('load', () => {    
    })
    img.addEventListener('error', () => {    
    })
    img.crossOrigin = 'anonymous'
    img.src = imageUrl        
```

* 手机端拍照上传，照片转向: 找了一个第三方库 [exif.js](https://github.com/exif-js/exif-js)， 可以判断， 照片是那个方向拍的, 知道是那个方向拍的，我们就可以旋转到正确的方向了

* 图片过大，无法上传: 这个就简单了，直接使用canvas的 drawImage就可以了

* ios 和 安卓下input file控件, 选择拍照项，然后取消的表现差异:ios下事件处理函数不会执行， 安卓下会执行，且事件对象里面file对象也存在, 只不过size为0

```javascript
 let file = event.target.files[0] || event.dataTransfer.files[0]
    let me = this
    // 由于安卓选择拍照, 取消也会有file实例但size为0; 所以可以在这里做一下判断
    if (file && file.size > 0) {
        me.uploadTip = true
    }
```

#### 结束语
>  这里只说了绘图的常见问题，以及上传图片的一些问题，其他问题我也想不起来了 。 知道的同学回顾下，不知道的就记一下。以后碰到了，就知道该怎么做了。这个活动花的时间比较长，主要在解决各种问题。
> HTML的表单元素，在PC端和移动端的差异还是蛮大的，而且ios和安卓的差异也大,如果没有实践，很难在开发前预测问题

canvas基础资料: [https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial)
