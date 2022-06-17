## 一、表单增强

```html
<form action="" method="get">
  <p>邮箱标签：<input type="email"></p>
  <p>数字标签：<input type="number"></p>
  <p>滑动条标签：<input type="range"></p>
  <p>搜索框标签：<input type="search"></p>
  <p>日期框标签：<input type="date"></p>
  <p>星期标签：<input type="week"></p>
  <p>月份标签：<input type="month"></p>
  <p>颜色标签：<input type="color"></p>
  <p>网址标签：<input type="url"></p>
</form>
```



![2](F:\study\文章积累\本地文章\interview-question\HTML\asset\2.png)



## 二、音视频标签

在h5之前，网页中内嵌音视频普遍会采用 flash 实现

```html
<video width="300" height="200" controls="controls">
	<source src="movie.mp4" type="video/mp4" />
</video>
```



## 三、Canvas + SVG

canvas 元素使用 js 在网页上绘制图像。

画布是一个矩形，我们可以控制其每一个像素。

canvas 拥有多种绘制路径、矩形、原型、字符以及添加图像的方法

```html
<canvas id="myCanvas" width="200" height="200"></canvas>
<script>
	var canvas = document.querySelector("#myCanvas");
  var ctx = canvas.getContext("2d");
  ctx.fillStyle = "#FF0000";
  ctx.fillRect(0, 0, 150, 75);
</script>
```



## 四、拖拽

在目标元素设置 `ondrop` 和 `ondropover` 方法，拖拽元素设置 `draggable="true"` 即可

```html
<body>
  <div id="div" ondrop="drop(event)" ondropover="dropover(event)"></div>
  <br />
  <img id="drag" src="img.png" draggable="true" ondragstart="dragstart(event)" />
</body>
```



## 五、本地存储

通过本地存储（LocalStorage），web 应用程序能够在用户浏览器中对数据进行本地存储。

在 `HTML5` 之前，应用程序数据只能存储在 `cookie` 中，包括每个服务器请求。本地存储则更安全，并且可以在不影响网站性能的前提下将大量数据存储于本地。

与 `cookie` 不同，存储限制要打的多（至少5MB），并且信息不会被传输到服务器。

本地存储经由起源地（origin），经由域和协议。所有页面，从起源地，能够存储和访问相同的数据。



## 六、地理位置

**地理位置（Geolocation）**是 `HTML5` 的重要特性之一，提供了确定用户位置的功能，借助这个特性我们能够开发基于位置信息的应用。

需要注意的是：PC 很多浏览器对于 `HTML5` 的定位技术是不太友好的，很多浏览器都是默认拒绝定位；相反在手机访问的时候，由于一般手机上都有 `GPS` 模块，所以定位效果会好很多。

```html
<script>
  function onSuccess(position) {
    console.log(position)
  }
  function onError(error) {
    console.error(error)
  }
	function getLocation() {
    const options = {
      enableHighAccuracy: true,
      maximumAge: 1000
    }
    console.log('获取位置信息开始--------->')
    if (navigator.geolocation) {
			// 走到这里说明，浏览器支持geolocation，参数里有两个回调函数，一个是定位成功后的处理操作，一个是定位失败后的处理操作
      navigator.geolocation.getCurrentPosition(onSuccess, onError, options);
		} else {
			// 否则浏览器不支持geolocation
			alert('您的浏览器不支持地理位置定位！');
		}
  }
  getLocation()
</script>
```






