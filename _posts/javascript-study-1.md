---
title: javascript学习笔记1
date: 2018-06-14 19:05:17
categories:
- 笔记
tags: 
- html
- javascript
- web
---
#### 变量

null 用于对象, undefined 用于变量，属性和方法。

对象只有被定义才有可能为 null，否则为 undefined。

undefined：表示变量未赋值，未定义的类型

null：用于清空的对象值, 类型是object


```javascript
<script>
	var person;//undefined
	person = {name:"bob", phone:"1344444"};
	person = null;
	typeof person;//object
	person = undefined;//typeof person = undefined
    var h = {name:"lisi", age:29};
    console.log(h.name);
    console.log(h[name]);
    var arr =['a', 3. 'hello', true;]
    alert(arr);
</script>
```
#### 运算符
```javascript
<script>
//一旦遇到非数值型的，后面一律字符串拼接
    console.log(2+3+4+'haha'+5+6)//9haha56

    //js中逻辑运算返回的是最早能判断表达式结果的那个值
    var a = false
    var b = 6
    var c = true;
    console.log(a || b || c) //输出6
    console.log(a && b && c)     //输出false
</script>
```

#### 控制结构

```javascript
var obj = {name:'lisi', age:29, area:'cn'}
for(var k in obj) {
    console.log(k + "~" + obj[k]);
}
```

#### 普通对象操作

- 属性
- 方法
- 静态方法

```javascript
var dt = new Date()
var str = "hell"
str.length
Math.random() *5 + 5 //5 ~ 10
```

#### Window对象操作
```javascript
//浏览器对象
window.alert()
window.confirm()
//window子对象-如判断浏览器是否支持cookie
window.navigator.cookieEnabled
//window地址栏子对象,如跳转地址栏
window.location.href = 'http://www.baidu.com';
//window子对象-历史记录
window.history.forward();
window.history.back();
//屏幕对象
window.screen
//DOM对象-页面的文档
window.docment

```
#### 判断对象
"john".constructor返回构造函数，如String()  { [native code] }

``` html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>判断是否为数组。</p>
<p id="demo"></p>
<script>
var fruits = ["Banana", "Orange", "Apple", "Mango"];
document.getElementById("demo").innerHTML = isArray(fruits);
function isArray(myArray) {
    return myArray.constructor.toString().indexOf("Array") > -1;
}
</script>

</body>
</html>
```
#### 变量的作用域
var是声明一个局部变量
不加var是赋值，会污染全局
首先在函数内部找，找不到会一直找最外层的变量

```javascript
window.str = 'global';
function t1() {
    function t2() {
        str = 'b'
    }
    t2();
}
t1();
alert(window.str);//全局被污染，此时str=b
```

#### DOM对象例子
```html
<!DOCTYPE html>
<html>
<head>
<meta charset='utf-8'>
<title>第一个特效</title>
<style>
	#test1{
		width:300px;
		height:200px;
		background:red;
	}
	#test2{
		width:300px;
		height:200px;
		background:blue;
	}
</style>
<script>
	function change(){
		var div = document.getElementById('test1');
		div.id = 'test2';
	}
</script
</head>

<body>
    <h1>特效是什么</h1>
	<div id='test1' onclick='change()'></div>
</body>
</html>
```
#### 找到DOM对象

```javascript
//根据Id来找，返回dom对象
document.getElementById('test');
//根据标签来找-返回对象的集合
document.getElementsByTagName('p');
//对于表单元素，用name来查找,返回对象集合，比如文字框，按钮
document.getElementsByName('username');
//按照类名查找，返回对象集合
document.getElementsByClassName('classname');
//找子节点，包含空白节点
document.getElementById('test').childNodes
//非标准属性，但兼容性好，不包含空白节点
document.getElementById('test').children

//父节点
document.getElementsByTagName('p')[2].parentNode;
```

#### 操作对象属性
以img为例

```html
<img src="a.jpg" alt='' title='1' style='width:200px;height:200px;'>
```
- object.src 标签属性  
- object.alt 标签属性 
- object.title 标签属性
- object.style 对象属性
    object.style.widith

*css属性中带有横线的，用大写字母替代
如border-top-style
用obj.style.borderTopStyle*

*class标签的属性为.className*

示例：
```html
<!DOCTYPE html>
<html>
<head>
<meta charset='utf-8'>
<title>第二个特效</title>
<style>
	.test1{
		background:red;
	}
	.test2{
		background:green;
	}
</style>
<script>
	function change(){
		var div = document.getElementsByTagName('div')[0];
		if (div.className.indexOf('test1') >=0){
			div.className = 'test2';
		}else if (div.className.indexOf('test2') >=0){
			div.className = 'test1';
		}
		div.style.width = parseInt(div.style.width) + 5 + 'px';
		div.style.height = parseInt(div.style.height) + 5 + 'px';
		div.style.borderBottomWidth = parseInt(div.style.borderBottomWidth) + 1 + 'px'
		}
</script>
</head>

<body>
	<div class='test1' onclick='change()' style='width:200px;height:200px;border-bottom:1px solid black;'></div>
</body>
</html>
```
#### 获取运行时的style对象
获取的属性是只读的
```
//IE9以下使用obj.currentStyle
function getStyle(obj, attr) {
    return obj.currentStyle? obj.currentStyle[attr] :window.getComputedStyle(obj, null)[attr];
}
```
因此之前的可以将style写在css中了
```html
<!DOCTYPE html>
<html>
<head>
<meta charset='utf-8'>
<title>第二个特效</title>
<style>
	div{
		width:200px;
		height:200px;
		border-bottom:1px solid black;
	}
	.test1{
		background:red;
	}
	.test2{
		background:green;
	}
</style>
<script>
	function getStyle(obj, attr) {
		return obj.currentStyle? obj.currentStyle[attr] :window.getComputedStyle(obj, null)[attr];
	}
	function change(){
		var div = document.getElementsByTagName('div')[0];
		if (div.className.indexOf('test1') >=0){
			div.className = 'test2';
		}else if (div.className.indexOf('test2') >=0){
			div.className = 'test1';
		}
		div.style.width = parseInt(getStyle(div, 'width')) + 5 + 'px';
		div.style.height = parseInt(getStyle(div, 'height')) + 5 + 'px';
		div.style.borderBottomWidth = parseInt(getStyle(div, 'borderBottomWidth')) + 1 + 'px'
	}
</script>
</head>

<body>
	<div class='test1' onclick='change()'></div>
</body>
</html>
```
#### 删除节点
1. 找到对象childobj
2. 找到父对象parentobj
3. parentobj.removeChild(childobj)

示例
```html
<!DOCTYPE html>
<html>
<head>
<meta charset='utf-8'>
<title>第三个特效</title>
<script>
	function delLast(){
		var lis = document.getElementsByTagName('li');
		var lastLi = lis[lis.length-1];
		lastLi.parentNode.removeChild(lastLi);
	}
</script>
</head>

<body>
<input type='button' value='del last' onclick='delLast()'/>
<ul>
	<li>春</li>
	<li>夏</li>
	<li>秋</li>
	<li>冬</li>		
</ul>
</body>
</html>
```
#### 创建节点
1. 创建对象obj
2. 找到父对象 parentObj
3. parentObj.appendChild

```javascript
function add() {
    var li = document.createElement('li');
    var txt = document.createTextNode('冬');
    li.appendChild(txt);

    var parent = document.getElementsByTagName('ul')[0];

    parent.appendChild(li);
}

```
#### 暴力操作节点
innerHTML

```javascript
val ul = document.getElementsByTagName('ul')[0];
ul.innerHTML = '<li>春</li><li>夏</li>'';
```
#### 联动菜单
事件+DOM操作

```html
<!DOCTYPE html>
<html>
<head>
<meta charset='utf-8'>
<title>联动菜单</title>
<script>
	var area =[['郑州' , '信阳' , '洛阳'], ['南京','无锡','苏州']]
	function ld(){
		var sel = document.getElementById('province');
		var selcity = document.getElementById('city');
			
		var  opt = '';
		if (sel.value < 0) {
			selcity.innerHTML = opt;
			return;
		}
		for (var i = 0, len = area[sel.value].length; i < len; i++) {
			opt = opt + "<option value = ' + i + '>" + area[sel.value][i] + '</option>';
		}
		selcity.innerHTML = opt;
	}
</script>
</head>

<body>
<select id='province' onchange='ld();'>
	<option value='-1'>请选择</option>
	<option value='0'>河南</option>
	<option value='1'>江苏</option>
</select>
<select id='city'/>
</body>
</html>
```







