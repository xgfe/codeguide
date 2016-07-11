# 项目最佳实践
## 1 前言
这是在团队在对各个管理进行review之后整理出的最佳实践，也是曾经踩过的一些坑或者没有做好的地方，希望大家多多总结，写出更加健（niu）壮（bi）的代码。

## 2 内容

### map和forEach的异同
`map`和`forEach`的共同点在于都是一种“遍历”数组的方法，forEach是**让数组的每一个元素都执行一次给定的函数**，但map更强调的是**对数组的每一个元素调用指定方法后的返回值组成的新数组**，重点在对数组中元素执行方法之后的新数组，而不是数组本身。  
比如要获取一个对象数组中的每一个对象的id并**组成一个新的数组**，这种情况下map就非常合适。

```
let idArr = objArr.map(obj => obj.id);
```
同`forEach`同样可以办到，如

```
let idArr = [];
objArr.forEach((obj) => {
	idArr.push(obj.id);
});
```
从上述代码中可以看出，这种场景下，`map`比`forEach`就更加简洁。但是，又有一种情况是需要注意的，如获取一个对象数组中的对象id是奇数的对象id并**组成一个新的数组**。使用forEach如下

```
let idArr = [];
objArr.forEach((obj) => {
    obj.id % 2 && idArr.push(obj.id);
});
```
用map的话好像是如下的代码

```
let idArr = objArr.map(obj => {
	obj.id % 2 && return obj.id;
});
```
但是这样是有问题的，因为`map`把对**每一个**元素执行函数之后的**返回结果**组成一个数组，当id是奇数的时候确实可以获取到对应的id，但是当id为偶数的时候，同样会有一个值被函数返回，就是`undefined`，即原数组中id为偶数的在新数组中都是`undefined`，而这却并不是我们要的结果。  
综上，`map`和`forEach`有各自的使用场景，实际项目中根据不同的需求而定，如果是纯遍历场景，建议使用`forEach`。

### 优雅的条件嵌套
嵌套的条件判断会导致方法的正常执行路径不明晰，使代码可读性下降。条件嵌套合理不合理读起来完全是两种不同的代码有木有。有一篇很好的文章可以参考：[嵌套条件的重构](http://efe.baidu.com/blog/replace-nested-conditional-with-guard-clauses/)

### 合理使用switch

和`if`一样，`switch`语句使用得不合理也会使代码非常糟糕，造成代码冗长，可读性差等问题，如下：

```
function badSwitchUsage(input){
	let week = '';
	switch(input){
		case 'Mon':
			week = '星期一';
			break;
		case 'Tues':
			week = '星期二';
			break;
		case 'Wed':
			week = '星期三';
			break;
		case 'Thur':
			week = '星期四';
			break;
		case 'Fri':
			week = '星期五';
			break;
		case 'Sta':
			week = '星期六';
			break;
		case 'Sun':
			week = '星期日';
			break;
		default:
			break;
	}
	return week;
}
```
看了上述代码估计很多人都要疯了。。。上述代码非常冗长，但是实现的功能却配不上这么长的代码，归根结底就是`switch`使用不对，如果非要使用`switch`，在上述代码中加入`return`会更好一些，

```
function badSwitchUsage(input){
	let week = '';
	switch(input){
		case 'Mon':
			return '星期一';
		case 'Thes':
			return '星期二';
		
		....
}		
```
当然，这种情况使用mapping更好一点

```
const mapping = {
	Mon:'星期一',
	Thes:'星期二',
	Wed:'星期三',
	Thur:'星期四',
	Fri:'星期五',
	Sta:'星期六',
	Sun:'星期日'
};
let goodCode = input => mapping[input];
```
瞬间清爽很多有木有！！

### 慎用toFixed进行四舍五入
`toFixed`可以进行四舍五入，但是有些情况下会有问题，如

```
1.35.toFixed(1) // 1.4 正确
1.335.toFixed(2) // 1.33  错误
1.3335.toFixed(3) // 1.333 错误
1.33335.toFixed(4) // 1.3334 正确
1.333335.toFixed(5)  // 1.33333 错误
1.3333335.toFixed(6) // 1.333333 错误
```

### 项目样式保证统一
select和ui-select尽量不要混用，统一使用ui-select

### 不要使用固定宽度布局
在非必要场景下，不要使用固定宽度布局，尽量使用自适应布局。

### 简化HTML标签嵌套层次
html文件中的标签能够合并就合并，减少冗余的代码

```
// bad code
<div class="warpper">
	<div class="inner">
		<div class="text-center">
			<div class="bg-dark text-bold">
				....
			</div>
		</div>
	</div>
</div>
```
部分html标签不是必须的，去掉的话可以使结构看上去更加清晰

```
<div class="warpper">
	<div class="inner text-center bg-dark text-bold">
		...
	</div>
</div>
```

### 模块中子模块的目录结构
我们项目中模块的划分比较清晰，即逻辑（js）、样式（css）和模板（html），如果模块只有一级的话这样划分是没有问题的，但是当模块中含有二级模块的话，就需要在在其中多加入一层目录结构，以此类推。

```
|-- main-module
	|-- tpl/   // main module modal template
	|-- child-module
		|-- tpl/ // child module modal template
		|-- childModuleCtrl.js
		|-- childModuleService.js
		|-- childModule.html
	|-- mainModuleCtrl.js
	|-- mainModuleService.js
	|-- mainModule.html

```

### `setTimeout` VS `setInterval`
`setTimeout`表示在指定的延迟时间之后调用一个函数或执行一个代码片段，`setInterval`表示周期性地调用一个函数或者执行一段代码，区别显而易见，但是在某些情况下，setTimeout也可以实现`setInterval`的功能，如

```
function foo(){
	console.log('foo');
	setTimeout(foo,1000);
}
setTimeout(foo,1000);
```
对于周期性执行函数的需求，使用`setInterval`需要额外注意，因为`setInterval`有时会因为其执行的内容而不能保证完全按照设置的时间间隔执行，准确点说可能比设置的时间间隔长，更加详细的说明可以看博文：[你真的了解setTimeout和setInterval吗？](http://qingbob.com/difference-between-settimeout-setinterval/)。

### 多个tab页中分页的使用
如果一个模块中有多个tab标签页，并且每一个tab中都有分页器，这种情况下需要特殊注意分页的设置，如果多个tab使用同一个分页器的话，当切换分页的时候对分页要重置，不然多个tab获取分页的时候会错乱。或者每一个tab使用自己的分页。

### 灵活使用鼠标样式
在可点击元素上设置鼠标样式为`cursor: pointer`，这样用户就知道这是一个可点击的元素。同理，可移动元素设置为`cursor: move`，其他交互状态同样如此。

