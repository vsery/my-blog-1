---
title: 使用js编写一个简单的web端框选功能
date: 2017-10-04
categories: web
tags:
  - javascript
  - region
---
今天我们来聊一下怎么使用原生javascript编写一个简单的框选功能。

# 需求描述

- 鼠标左键按下不放，移动鼠标出现矩形选框；

- 鼠标左键松开，根据上边出现的矩形选框统计选框范围内的DOM元素；

嗯...上边的功能描述看着是挺简单的，但实现起来也还是会有些地方需要斟酌思考的。比如，如果我们的框选范围不是`document.body`，而是某一个`div`里边进行框选呢？而现实开发过程中，我们会遇上的应该就是第二种情况。

# 怎么实现

二话不说，咱们动手写代码吧！因为更好的兼容性，这里就避免了一些ES6的语法，如果是用的其他框架来写的话，代码上相应的也要做一些调整。

```html
<head>
<style>
.fileDiv {
  display: inline-block;
  width: 100px;
  height: 100px;
  margin: 24px;
  background-color: blue;
}
</style>
</head>
<body>
  <div class="fileDiv"></div>
  <div class="fileDiv"></div>
  <div class="fileDiv"></div>
  <div class="fileDiv"></div>
  <div class="fileDiv"></div>
  <div class="fileDiv"></div>
  <div class="fileDiv"></div>
  <div class="fileDiv"></div>
</body>
```

## 添加鼠标事件监听

由于js自身并没有带有鼠标点击按住不放这样子的事件，这里我们不仅需要检测鼠标左键点击按下，还要加一个定时器来检测鼠标是否按住不放了。

```js
<script>
  (function () {
    // 定时器id
    var mouseStopId;
    // 是否开启框选功能
    var mouseOn = false;
    // 用来存放鼠标点击初始位置
    var startX = 0;
    var startY = 0;
    // 添加鼠标按下监听事件
    document.body.addEventListener('mousedown', function (e) {
      // 阻止事件冒泡
      clearEventBubble(e);
      // 判断是否为鼠标左键被按下
      if (e.buttons !== 1 || e.which !== 1) return;

      mouseStopId = setTimeout(function () {
        mouseOn = true;
        startX = e.clientX;
        startY = e.clientY;
      }, 300); // 间隔300毫秒后执行，判定这时候鼠标左键被按住不放
    });

    // 添加鼠标移动事件监听
    document.body.addEventListener('mousemove', function (e) {
      // 如果并非框选开启，退出
      if (!mouseOn) return;
      // 阻止事件冒泡
      clearEventBubble(e);
      // 处理鼠标移动
      // codes
    });

    // 添加鼠标点击松开事件监听
    document.body.addEventListener('mouseup', function (e) {
      // 阻止事件冒泡
      clearEventBubble(e);
      // 处理鼠标点击松开
      // codes
    });

    function clearEventBubble (e) {
      if (e.stopPropagation) e.stopPropagation();
      else e.cancelBubble = true;

      if (e.preventDefault) e.preventDefault();
      else e.returnValue = false;
    }
  })();
</script>
```

## 添加框选可视化元素

[框选可视化示意图](http://upload-images.jianshu.io/upload_images/1626912-4935e189ce4098e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们有了事件监听还不够，为了更好的交互效果，我们需要一个随时跟随着鼠标移动的框选框元素，用于让用户随时感知框选范围。

```js
<script>
  (function () {
    var mouseStopId;
    var mouseOn = false;
    var startX = 0;
    var startY = 0;
    document.body.addEventListener('mousedown', function (e) {
      clearEventBubble(e);
      if (e.buttons !== 1 || e.which !== 1) return;

      mouseStopId = setTimeout(function () {
        mouseOn = true;
        startX = e.clientX;
        startY = e.clientY;
        // 创建一个框选元素
        var selDiv = document.createElement('div');
        // 给框选元素添加css样式，这里使用绝对定位
        selDiv.style.cssText = 'position:absolute;width:0;height:0;margin:0;padding:0;border:1px dashed #eee;background-color:#aaa;z-index:1000;opacity:0.6;display:none;';
        // 添加id
        selDiv.id = 'selectDiv';
        document.body.appendChild(selDiv);
        // 根据起始位置，添加定位
        selDiv.style.left = startX + 'px';
        selDiv.style.top = startY + 'px';
      }, 300);
    });

    document.body.addEventListener('mousemove', function (e) {
      if (!mouseOn) return;
      clearEventBubble(e);
      // 获取当前坐标
      var _x = e.clientX;
      var _y = e.clientY;
      // 根据坐标给选框修改样式
      var selDiv = document.getElementById('selectDiv');
      selDiv.style.display = 'block';
      selDiv.style.left = Math.min(_x, startX) + 'px';
      selDiv.style.top = Math.min(_y, startY) + 'px';
      selDiv.style.width = Math.abs(_x - startX) + 'px';
      selDiv.style.height = Math.abs(_y - startY) + 'px';
      // 如果需要更直观一点的话，我们还可以在这里进行对框选元素覆盖到的元素进行修改被框选样式的修改。
    });

    document.body.addEventListener('mouseup', function (e) {
      clearEventBubble(e);
    });

    function clearEventBubble (e) {
      if (e.stopPropagation) e.stopPropagation();
      else e.cancelBubble = true;

      if (e.preventDefault) e.preventDefault();
      else e.returnValue = false;
    }
  })();
</script>
```

## 添加鼠标松开事件监听

[元素是否被选中示意图](http://upload-images.jianshu.io/upload_images/1626912-7af06a966948fc67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们没有在鼠标移动的时候去实时统计被框选到的DOM元素，如果需要实时统计或者实时修改被选择的DOM元素的样式，以便更准确的让用户感知到被框选的内容的话，可以选择在`mousemove`事件里边去实现以下代码：

```js
<script>
  (function () {
    var mouseStopId;
    var mouseOn = false;
    var startX = 0;
    var startY = 0;
    document.onmousedown = function (e) {
      clearEventBubble(e);
      if (e.buttons !== 1 || e.which !== 1) return;

      mouseStopId = setTimeout(function () {
        mouseOn = true;
        startX = e.clientX;
        startY = e.clientY;
        var selDiv = document.createElement('div');
        selDiv.style.cssText = 'position:absolute;width:0;height:0;margin:0;padding:0;border:1px dashed #eee;background-color:#aaa;z-index:1000;opacity:0.6;display:none;';
        selDiv.id = 'selectDiv';
        document.body.appendChild(selDiv);
        selDiv.style.left = startX + 'px';
        selDiv.style.top = startY + 'px';
      }, 300);
    }

    document.onmousemove = function (e) {
      if (!mouseOn) return;
      clearEventBubble(e);
      var _x = e.clientX;
      var _y = e.clientY;
      var selDiv = document.getElementById('selectDiv');
      selDiv.style.display = 'block';
      selDiv.style.left = Math.min(_x, startX) + 'px';
      selDiv.style.top = Math.min(_y, startY) + 'px';
      selDiv.style.width = Math.abs(_x - startX) + 'px';
      selDiv.style.height = Math.abs(_y - startY) + 'px';
    };

    // 添加鼠标松开事件监听
    document.onmouseup = function (e) {
      if (!mouseOn) return;
      clearEventBubble(e);
      var selDiv = document.getElementById('selectDiv');
      var fileDivs = document.getElementsByClassName('fileDiv');
      var selectedEls = [];
      // 获取参数
      var l = selDiv.offsetLeft;
      var t = selDiv.offsetTop;
      var w = selDiv.offsetWidth;
      var h = selDiv.offsetHeight;
      for (var i = 0; i < fileDivs.length; i++) {
        var sl = fileDivs[i].offsetWidth + fileDivs[i].offsetLeft;
        var st = fileDivs[i].offsetHeight + fileDivs[i].offsetTop;

        if (sl > l && st > t && fileDivs[i].offsetLeft < l + w && fileDivs[i].offsetTop < t + h) {
          // 该DOM元素被选中，进行处理
          selectedEls.push(fileDivs[i]);
        }
      }
      // 打印被选中DOM元素
      console.log(selectedEls);
      // 恢复参数
      selDiv.style.display = 'none';
      mouseOn = false;
    };

    function clearEventBubble (e) {
      if (e.stopPropagation) e.stopPropagation();
      else e.cancelBubble = true;

      if (e.preventDefault) e.preventDefault();
      else e.returnValue = false;
    }
  })();
</script>
```

这里判断一个元素是否被选中采用的判断条件是：

- 该DOM元素的最右边（`fileDiv[i].offsetLeft + fileDiv[i].offsetWidth`）是否要比选框元素最左边（`selDiv.offsetLeft`）的位置要小；

- 该DOM元素的最下边（`fileDiv[i].offsetTop + fileDiv[i].offsetHeight`）是否要比选框元素的最上边（`selDiv.offsetTop`）的位置要大；

- 该DOM元素的最左边（`fileDiv[i].offsetLeft`）是否要比选框元素的最后边(`selDiv.offsetLeft + selDiv.offsetWidth`)的位置数值要小；

- 该DOM元素的最上边（`fileDiv[i].offsetTop`）是否要比选框元素的最下边（`selDiv.offsetTop + selDiv.offsetHeight`）的位置数值要小；

满足了以上四个条件，即可判别为该DOM元素被选中了。

## 实际应用

上边的例子，举得有些过于简单了。实际的开发当中，框选的范围往往不可能是整个`document.body`，而是某一个具体的有特定宽度跟高度限制的元素。这个时候，就还需要考虑这个框选容器元素造成的定位偏差，以及容器内元素过多，出现滚动条的情况了。

乍一看，上边的情况需要考虑的因素多了不少，比较容易乱。我这里采用的方法是修改坐标系的方式来实现上边描述的功能。上文我们已经实现了在`document.body`整个页面左上角顶点作为坐标原点来实现框选功能，这时候我们需要修改坐标原点为框选容器的左上角顶点作为坐标原点即可。

换言之，就是修改`mousedown`跟`mousemove`事件时，初始位置由原来的`e.clientX`跟`e.clientY`修改为`e.clientX - selectContaienr.offsetLeft + selectContainer.scrollLeft`跟`e.clientY - selectContainer.offsetTop + selectContainer.scrollTop`。

[坐标更改示意图](http://upload-images.jianshu.io/upload_images/1626912-5c8b631b43846fb8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```html
<html>
  <head>
    <title>region</title>
    <style>
      body {
        margin: 0;
        padding: 0;
      }
      #selectContainer {
        position: relative;
        width: 400px; /* 演示宽高与位置 */
        height: 400px;
        top: 200px;
        left: 200px;
        border: 1px solid #eee;
        overflow: hidden;
        overflow-y: auto;
      }
      .fileDiv {
        display: inline-block;
        width: 100px;
        height: 100px;
        margin: 24px;
        background-color: #0082CC;
      }
    </style>
  </head>
  <body>
    <div id="selectContainer">
      <div class="fileDiv"></div>
      <div class="fileDiv"></div>
      <div class="fileDiv"></div>
      <div class="fileDiv"></div>
      <div class="fileDiv"></div>
      <div class="fileDiv"></div>
      <div class="fileDiv"></div>
      <div class="fileDiv"></div>
      <div class="fileDiv"></div>
      <div class="fileDiv"></div>
      <div class="fileDiv"></div>
      <div class="fileDiv"></div>
      <div class="fileDiv"></div>
      <div class="fileDiv"></div>
    </div>
  </body>
</html>
```

```js
<script>
  (function () {
    var mouseStopId;
    var mouseOn = false;
    var startX = 0;
    var startY = 0;
    document.onmousedown = function (e) {
      clearEventBubble(e);
      if (e.buttons !== 1 || e.which !== 1) return;

      mouseStopId = setTimeout(function () {
        mouseOn = true;
        // 获取容器元素
        var selectContainer = document.getElementById('selectContainer');
        // 调整坐标原点为容器左上角
        startX = e.clientX - selectContainer.offsetLeft + selectContainer.scrollLeft;
        startY = e.clientY - selectContainer.offsetTop + selectContainer.scrollTop;
        var selDiv = document.createElement('div');
        selDiv.style.cssText = 'position:absolute;width:0;height:0;margin:0;padding:0;border:1px dashed #eee;background-color:#aaa;z-index:1000;opacity:0.6;display:none;';
        selDiv.id = 'selectDiv';
        // 添加框选元素到容器内
        document.getElementById('selectContainer').appendChild(selDiv);
        selDiv.style.left = startX + 'px';
        selDiv.style.top = startY + 'px';
      }, 300);
    }

    document.onmousemove = function (e) {
      if (!mouseOn) return;
      clearEventBubble(e);
      var selectContainer = document.getElementById('selectContainer');
      var _x = e.clientX - selectContainer.offsetLeft + selectContainer.scrollLeft;
      var _y = e.clientY - selectContainer.offsetTop + selectContainer.scrollTop;
      // 鼠标移动超出容器内部，进行相应的处理
      // 向下拖拽
      if (_y >= _H && selectContainer.scrollTop <= _H) {
        selectContainer.scrollTop += _y - _H;
      }
      // 向上拖拽
      if (e.clientY <= selectContainer.offsetTop && selectContainer.scrollTop > 0) {
        selectContainer.scrollTop = Math.abs(e.clientY - selectContainer.offsetTop);
      }
      var selDiv = document.getElementById('selectDiv');
      selDiv.style.display = 'block';
      selDiv.style.left = Math.min(_x, startX) + 'px';
      selDiv.style.top = Math.min(_y, startY) + 'px';
      selDiv.style.width = Math.abs(_x - startX) + 'px';
      selDiv.style.height = Math.abs(_y - startY) + 'px';
    };

    document.onmouseup = function (e) {
      if (!mouseOn) return;
      clearEventBubble(e);
      var selDiv = document.getElementById('selectDiv');
      var fileDivs = document.getElementsByClassName('fileDiv');
      var selectedEls = [];
      var l = selDiv.offsetLeft;
      var t = selDiv.offsetTop;
      var w = selDiv.offsetWidth;
      var h = selDiv.offsetHeight;
      for (var i = 0; i < fileDivs.length; i++) {
        var sl = fileDivs[i].offsetWidth + fileDivs[i].offsetLeft;
        var st = fileDivs[i].offsetHeight + fileDivs[i].offsetTop;

        if (sl > l && st > t && fileDivs[i].offsetLeft < l + w && fileDivs[i].offsetTop < t + h) {
          selectedEls.push(fileDivs[i]);
        }
      }
      console.log(selectedEls);
      selDiv.style.display = 'none';
      mouseOn = false;
    };

    function clearEventBubble (e) {
      if (e.stopPropagation) e.stopPropagation();
      else e.cancelBubble = true;

      if (e.preventDefault) e.preventDefault();
      else e.returnValue = false;
    }
  })();
</script>
```

## 使用前端框架

上边的代码，我们只是在一个html文件里边实现了框选的功能。很多时候，我们会使用一些前端框架来编写框选的功能（例如`vue.js`，`angular`,`react`,`polymer`之类的前端框架）。这个时候，我们可以利用框架自身的生命周期的函数，添加对应的监听事件，然后在`mouseup`事件里移除掉上边这些事件监听，以减少不必要的资源消耗。而且，很多时候，组件化的使用，使得被框选的元素，往往也是一个可重复利用的小组件，也是需要根据相应的框架的对应的途径获取到对应的DOM元素来获取其属性。

## 完整例子查看

[查看完整代码](https://github.com/classLfz/my-blog/blob/master/examples/mouse-region.html)