---
title: 使用原生JS编写一个图片裁剪器
date: 2017-07-23 22:35:00
categories: web前端
tags:
  - web前端
  - canvas
---
> 作为一个前端开发人员，这次的这篇博客文章终于“正常”了。

这应该算是一个造轮子的实践，JS的图片开源裁剪器有很多，像使用JQuery库编写的`copper`插件很多，在github上边的star数量也不少，现流行的前端框架也肯定有对应的图片裁剪器，都是可以选择的成熟的技术方案。但不是所有的情况都能适用，设计师的要求，本身项目的条件限制等等原因，有些时候，还是“自己动手，丰衣足食”啊！

**因为用到了很多`html5`的特性，所以，这里编写的图片裁剪器，只能是适合于支持这些HTML5特性的浏览器才能够正常的使用。**

## 需求描述

- 点击按钮选择图片，并进行展示；

- 展示区有一个正方形裁剪区，图片可以放大缩小；

- 在展示区，点击鼠标后拖拽，可对图片的位置进行调试，但展示图片不能越过裁剪区；

- 点击裁剪按钮，对位于裁剪区的图片进行裁剪，并展示裁剪后的图片；

## HTML & CSS部分

首先，我们要编写一些基础的HTML代码：

```html
<div id="scissorsContainer"> 
  <input id="imageInput" type="file" hidden onchange="ImageInputChanged(event)">

  <label class="btn" for="imageInput">SELECT IMAGE</label>

  <div id="workArea" onmousedown="startDrag(event)">
    <div id="overlay">
      <div id="overlayInner"></div>
      <img id="avatorImg" onload="avatorImgChanged()" alt="">
    </div>
  </div>

  <div id="resizeBox">
    <div>
      <button class="btn" onclick="resizeDown()">缩小</button>
      <button class="btn" onclick="resizeUp()">放大</button>
    </div>
  </div>

  <button class="btn" onclick="crop()">CROP</button>
</div>
<h4>Image Show</h4>
<img id="imageShow" src="" alt="">
```

添加样式。我这里是用了flex布局，如果不想用flex布局的话，可以选择其他的布局方式。

```css
body {
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
}
#scissorsContainer {
  display: flex;
  justify-content: center;
  flex-direction: column;
}
.btn {
  padding: 8px;
  font-size: 14px;
  border: 1px solid #1976d2;
  border-radius: 2px;
  background: #ffffff;
  text-align: center;
  cursor: pointer;
}
.btn:hover {
  background-color: #1976d2;
  color: #ffffff;
}
#workArea {
  width: 500px;
  height: 500px;
  position: relative;
  margin: 16px 0;
  overflow: hidden;
}
#overlay {
  position: absolute;
  width: 100%;
  height: 100%;
  background: #eeeeee;
  display: flex;
  justify-content: center;
  align-items: center;
}
#overlayInner {
  width: 300px;
  height: 300px;
  border: 100px solid gray;
  opacity: 0.7;
  z-index: 1;
}
#avatorImg {
  width: auto;
  height: auto;
  border: none;
  outline: none;
  position: absolute;
}
#resizeBox {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 16px;
}
#resizeBox > div > button {
  margin: 0 8px;
}
#imageShow {
  width: 300px;
  height: 300px;
}
```

## 设置全局变量

这里，我们设置一些JS代码里边需要的全局变量，具体的作用，下边的部分我们会介绍到的。

```js
window.onload = function() {
  // *** 定义全局变量 ***
  // 全局需要的元素
  window.workArea = document.querySelector('#workArea');
  window.avatorImg = document.querySelector('#avatorImg');
  window.imageShow = document.querySelector('#imageShow');
  // 鼠标初始位置坐标数值
  window.mouseStartX = 0;
  window.mouseStartY = 0;
  // 图片初始化后的尺寸记录
  window.initLength = {
    width: 0,
    height: 0
  };
  // 图片原始尺寸记录
  window.primitiveLength = {
    width: 0,
    height: 0
  };
  // 图片放大缩小的数值
  window.resizeValue = 0;
  // 图片呈现高度&宽度，需要根据HTML以及CSS部分的overlayInner的高宽度一致
  window.showSide = document.querySelector('#overlayInner').clientWidth;
  // 裁剪的图片类型
  window.croppedImageType = 'image/png';
}
```

## 获取图片数据，并展示

我们在上边的HTML部分写了一个隐藏的`input[type="file"]`，通过一个`label`标签来触发它，同时监听这个`input`元素，当它的值发生改变时，可以通过一个`FileReader`的对象获取到这个`input`的数据，并将它转换成`base64`格式的数据。

```js
/**
  * 图片选择元素的值发生变化后，重置图片裁剪区的样式
  * @param {Object} e input数值变化事件
  */
function ImageInputChanged(e) {
  var file = e.target.files[0];
  var reader = new FileReader();
  reader.onload = function(event) {
    // 赋值给图片展示元素
    avatorImg.src = event.target.result;
    // 重置样式
    avatorImg.style.width = 'auto';
    avatorImg.style.height = 'auto';
    avatorImg.style.top = 'auto';
    avatorImg.style.left = 'auto';
  }

  reader.readAsDataURL(file);
}
```

## 图片展示区的数据发生变化

图片展示区的数据发生变化，我们在这里除了收集图片的原始信息以外，还对图片进行了初始化的像素调整。

```js
function avatorImgChanged() {
  if (avatorImg.offsetWidth >= avatorImg.offsetHeight) {
    avatorImg.style.top = '100px';
    initLength.width = showSide * avatorImg.offsetWidth / avatorImg.offsetHeight
    initLength.height = showSide;
  } else {
    avator.style.left = '100px';
    initLength.height = showSide * avatorImg.offsetWidth / avatorImg.offsetWidth;
    initLength.width = showSide;
  }
  // 保存新的图片原始像素值
  primitiveLength = {
    width: avatorImg.offsetWidth,
    height: avatorImg.offsetHeight
  };
  // 更新图片样式
  avatorImg.style.width = initLength.width + 'px';
  avatorImg.style.height = initLength.height + 'px';
}
```

## 图片的放大与缩小

在上边对应的HTML部分，我们添加了图片放大与缩小的按钮。

```js
/**
  * 图片放大
  */
function resizeUp() {
  if (resizeValue <= 0) return;
  resizeValue += 10;
  resize();
}

/**
  * 图片缩小
  */
function resizeDown() {
  resizeValue -= 10;
  resize();
}

/**
  * 修改图片比例大小
  */
function resize() {
  avatorImg.style.width = (resizeValue + 100) / 100 * initLength.width + 'px';
  avatorImg.style.height = (resizeValue + 100) / 100 * initLength.height + 'px';
}
```

## 图片的拖拽

图片的拖拽，就是利用鼠标的三个事件来完成——`mousedown`，`mousemove`以及`mouseup`。我们在`mousedown`的时候，记录鼠标的初始位置，添加`mousemove`以及`mouseup`事件监听函数。

```js
/**
  * 监测鼠标点击，开始拖拽
  * @param {Object} e 鼠标点击事件
  */
function startDrag(e) {
  e.preventDefault();
  if (avatorImg.src) {
    // 记录鼠标初始位置
    window.mouseStartX = e.clientX;
    window.mouseStartY = e.clientY;
    // 添加鼠标移动以及鼠标点击松开事件监听
    workArea.addEventListener('mousemove', window.dragging, false);
    workArea.addEventListener('mouseup', window.clearDragEvent, false);
  }
}
```

图片的拖拽，我们这里做了一些限制，限制不让它拖拽出裁剪区。这主要是防止我们裁剪出不属于原图的空白区域。

```js
/**
  * 处理拖拽
  * @param {Object} e 鼠标移动事件
  */
function dragging(e) {
  // *** 图片不存在 ***
  if (!avatorImg.src) return;
  
  // *** 图片存在 ***
  // X轴
  let _moveX = avatorImg.offsetLeft + e.clientX - mouseStartX;
  // 这里的100是HTML里边overlayInner的border属性的宽度
  // 下边的400就是overlayInner元素的边长加上border的宽度之和
  if (_moveX >= 100) {
    avatorImg.style.left = '100px';
    mouseStartX = e.clientX;
    return;
  } else if (_moveX <= 400 - avatorImg.offsetWidth) {
    _moveX = 400 - avatorImg.offsetWidth;
  }

  avatorImg.style.left = _moveX + 'px';
  mouseStartX = e.clientX;

  // Y轴
  let _moveY = avatorImg.offsetTop + e.clientY - mouseStartY;
  if (_moveY >= 100) {
    avatorImg.style.top = '100px';
    mouseStartY = e.clientY;
    return;
  } else if (_moveY <= 400 - avatorImg.offsetHeight) {
    _moveY = 400 - avatorImg.offsetHeight;
  }
  avatorImg.style.top = _moveY + 'px';
  mouseStartY = e.clientY;
}
```

## 裁剪

最后，我们利用HTML5的canvas元素具有的[`ctx.drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight);`](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/drawImage)方法，描绘出位于裁剪区的图片信息。

```js
/**
  * 对图片进行裁剪
  */
function crop() {
  if (!avatorImg.src) return;
  let _cropCanvas = document.createElement('canvas');
  // 计算边长
  let _side = (showSide / avatorImg.offsetWidth) * primitiveLength.width;
  _cropCanvas.width = _side;
  _cropCanvas.height = _side;
  // 计算截取时从原图片的原始长度的坐标
  // 因为图片有可能会被放大/缩小，这时候，初始化时记录下来的primitiveLength信息就有用处了
  let _sy = (100 - avatorImg.offsetTop) / avatorImg.offsetHeight * primitiveLength.height;
  let _sx = (100 - avatorImg.offsetLeft) / avatorImg.offsetWidth * primitiveLength.width;
  // 绘制图片
  _cropCanvas.getContext('2d').drawImage(avatorImg, _sx, _sy, _side, _side, 0, 0, _side, _side);
  // 保存图片信息
  let _lastImageData = _cropCanvas.toDataURL(croppedImageType);
  // 将裁剪出来的信息展示
  imageShow.src = _lastImageData;
  imageShow.style.width = showSide + 'px';
  imageShow.style.height = showSide + 'px';
}
```

## 例子

<a href="https://github.com/classlfz/my-blog/examples/write-an-image-scissors.html" download="write-an-image-scissors">下载例子</a>