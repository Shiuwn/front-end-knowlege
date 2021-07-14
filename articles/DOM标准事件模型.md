### 事件流

考虑事件传递中有两个嵌套元素，当点击内部的元素时，存在两种处理事件的方式：
1. 从外到内触发目标元素（Netscape Navigator的方式）
2. 从内到外触发目标元素（IE和其他浏览器使用的这种方式）

`W3C` 采取了折中的方式。根据 `W3C` 目标事件进入DOM树要经历三个阶段：
1. 捕获阶段：事件从根事件元素（root event target）传递到目标元素
2. target阶段：事件传递给了目标元素
3. 冒泡阶段（可选）：事件传递从目标元素传递到根事件元素。冒泡阶段仅仅发生在event.bubbles==true的时候。

### 事件处理模型

#### 1. DOM Level 0

这个模型是`Netscape Navigator`引入的，有两种类型：行内式和传统式。

##### 行内模型

```html
<!doctype html>
<html lang="en">
<head>
	<meta charset="utf-8">
	<title>Inline Event Handling</title>
</head>
<body>

	<h1>Inline Event Handling</h1>

	<p>Hey <a href="http://www.example.com" onclick="triggerAlert('Joe'); return false;">Joe</a>!</p>

	<script>
		function triggerAlert(name) {
			window.alert("Hey " + name);
		}
	</script>
</body>
</html>

```
对于行内式的处理模型普遍有一种错觉：它允许传入自定义的变量，也就是这里`triggerAlert(name)` 中的 `name`。但是事实上在`JavaScript 引擎`中发生的情况是，浏览器会创建一个匿名函数包裹`onclick`属性内的代码也就是

```js

function (){
    triggerAlert('Joe'); 
    return false;
}

```

>`NOTE`
>
> `DOM Level 0` 混合了`Netscape Navigator 3.0`和`IE 3.0` 的 `HTML` 的文档功能。

##### 传统模型

在传统模型中，事件处理可以在js中添加和删除，但是和行内模型一样，这个处理函数只能添加一个，下面的代码展示了添加事件处理函数和删除的方式：
```js
<!doctype html>
<html lang="en">
<head>
	<meta charset="utf-8">
	<title>Traditional Event Handling</title>
</head>

<body>
	<h1>Traditional Event Handling</h1>
	
	<p>Hey Joe!</p>

	<script>
		var triggerAlert = function () {
			window.alert("Hey Joe");
		};
		
		// Assign an event handler
		document.onclick = triggerAlert;
		
		// Assign another event handler
		window.onload = triggerAlert;
		
		// Remove the event handler that was just assigned
		window.onload = null;
	</script>
</body>
</html>
```

当需要为处理函数传递参数时，可以使用闭包的形式。

### DOM Level 2

`W3C`使用了更为灵活的事件处理模型：

- `addEventListener` 添加处理程序

- `removeEventListener` 删除处理程序

- `dispatchEvent` 分发事件给处理程序



同时还可以调用事件对象的方法，来干预事件的传递和默认事件的触发：

- `stopPropagation()` 阻止事件冒泡
- `preventDefault()` 阻止目标元素的默认动作执行

> `NOTE`

```
并不是所有的事件都可以阻止冒泡和取消默认动作
scroll 事件可以冒泡不可以取消
blur 和 focus 事件也是无法冒泡和取消的
Media 事件也不会冒泡，如视频和音频的播放事件
mouseleave和mouseenter 事件也不会冒泡，但是mouseout和mouseover会触发冒泡
另外，在传递的Event对象里面可以通过查看 bubbbles 属性知道这个事件是否是一个冒泡事件（true代表是冒泡事件），cancelable 是否可以取消这个事件，eventPhase 事件流的阶段，等等。

参考3 里面指明了哪些事件是可以冒泡，哪些事件是可以取消的
```

`DOM Level 2` 的事件处理模型允许对目标元素绑定多个事件，还可以指定在捕获或者冒泡阶段处理事件。

> 参考（部分直接翻译）
>
> 1. [DOM Events](https://en.wikipedia.org/wiki/DOM_events#Event_flow)
> 2. [JavaScript中那些不会冒泡的事件](https://zhuanlan.zhihu.com/p/164844013)
> 3. [Document Object Model Events](https://www.w3.org/TR/DOM-Level-2-Events/events.html#Events-overview)

