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

