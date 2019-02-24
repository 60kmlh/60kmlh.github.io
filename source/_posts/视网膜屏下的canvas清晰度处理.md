---
title: 视网膜屏下的canvas清晰度处理
date: 2017-09-01
tags: ['canvas','dpr']
categories: ['工作总结']
---
## 背景
最近使用图表插件在移动端绘制canvas图表的时候，发现画出来的图很模糊。经过排查和查询相关资料，发现和视网膜屏设备的设备像素比有关。
## 相关知识
### canvas的width 和 canvas的style.width
style.width是用来设定canvas元素在浏览器中被渲染的宽度（高度同理）
````html
<canvas style="border:1px solid;width:100px;height:100px;"></canvas>
````
<canvas style="border:1px solid;width:100px;height:100px;"></canvas>

width属性是用来设置canvas元素里面画布的宽度
````html
<canvas style="border:1px solid;" width="100px" height="100px"></canvas>
````
<canvas style="border:1px solid;" width="100px" height="100px"></canvas>

最终绘制出来的宽度 = 需要绘制的宽度 / 画布宽 * 画框宽

可以看到在不同元素宽，画布宽下绘制出来的（0，0）到（100，100）效果是不一样的。
<canvas id="diagonal1" style="border:1px solid;display:inline-block;margin-right:35px" width="100px" height="100px"></canvas><canvas id="diagonal2" style="border:1px solid;width:200px;height:200px;display:inline-block;margin-right:35px" width="100px" height="100px"></canvas><canvas id="diagonal3" style="border:1px solid;width:200px;height:200px;display:inline-block"></canvas>

### 视网膜屏幕
* 物理像素(physical pixel)

物理像素又被称为设备像素，他是显示设备中一个最微小的物理部件。每个像素可以根据操作系统设置自己的颜色和亮度。

* 设备独立像素(density-independent pixel)

设备独立像素也称为密度无关像素，可以认为是计算机坐标系统中的一个点，这个点代表一个可以由程序使用的虚拟像素(比如说CSS像素)，然后由相关系统转换为物理像素。

* CSS像素

CSS像素是一个抽像的单位，主要使用在浏览器上，用来精确度量Web页面上的内容。一般情况之下，CSS像素称为与设备无关的像素(device-independent pixel)，简称DIPs

* 设备像素比(device pixel ratio)

设备像素比简称为dpr，其定义了物理像素和设备独立像素的对应关系。它的值可以按下面的公式计算得到：

    设备像素比 ＝ 物理像素 / 设备独立像素

视网膜屏幕的dpr是大于1的，比如iphone6为2。

以iphone6为例，在上面绘制canvas，绘制1px x 1px时，实际上是使用了2物理像素 x 2物理像素，所以看起来会有模糊的效果。
## 解决思路
将画布先放大dpr倍，画框维持不变，根据公式：最终绘制出来的宽度 = 需要绘制的宽度 / 画布宽 * 画框宽，可知，此时绘制出来的图像会缩小dpr * dpr倍，因为图像缩小了来显示，这样看就是清晰的，但是图像被缩小了。

再调用canvas.scale(dpr, dpr),将canvas的绘图的值放大dpr * dpr倍，重新绘制,这样呈现出来的图像大小就是和原来一样，而且清晰的。此处理和视网膜屏幕下的1px处理是类似的。
## 代码
````javascript
var scaleFn = function(canvas, context, customWidth, customHeight) {
  if(!canvas || !context) { throw new Error('Must pass in `canvas` and `context`.'); }

  var width = customWidth ||
              canvas.width ||
              canvas.clientWidth;
  var height = customHeight ||
               canvas.height ||
               canvas.clientHeight;
  var deviceRatio = window.devicePixelRatio || 1;
  var ratio = deviceRatio;

  if (ratio !== 1) {
    canvas.width = Math.round(width * ratio);
    canvas.height = Math.round(height * ratio);
    canvas.style.width = width + 'px';
    canvas.style.height = height + 'px';
    context.scale(ratio, ratio);
  }
  return ratio;
};
````
## 参考资料

[https://www.html5rocks.com/en/tutorials/canvas/hidpi/](https://www.html5rocks.com/en/tutorials/canvas/hidpi/)
[https://www.npmjs.com/package/canvas-dpi-scaler](https://www.npmjs.com/package/canvas-dpi-scaler)
[https://bugs.chromium.org/p/chromium/issues/detail?id=277205](https://bugs.chromium.org/p/chromium/issues/detail?id=277205)

<script type="text/javascript">function drawDiagonal(id){var canvas=document.getElementById(id);
    var context=canvas.getContext("2d");
    context.beginPath();
    context.moveTo(0,0);
    context.lineTo(100,100);
    context.stroke();
}

    drawDiagonal("diagonal1");
    drawDiagonal("diagonal2");
    drawDiagonal("diagonal3");
</script>
