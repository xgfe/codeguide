# Angular项目编码规范
<a name="table-of-contents"></a> 
## 目录 

  1. [前言](#preface)
  2. [最好的代码实践](#best-practices)
  3. [Angular弃用的特性	](#deprecated-angular-features)
  4. [约定俗成](#conventions)
  5. [Angular封装](#angular-wrappers)
 
<a name="preface"></a>
## 1 前言
这是在[JavaScript规范](https://github.com/xgfe/codeguide/blob/master/javascript.md)基础上针对Angular项目的“扩展包”，规范了在使用Angular的时候需要注意的代码规范。  
使用的是[eslint-plugin-angular](https://github.com/Gillespie59/eslint-plugin-angular)这个插件，代码规范的基础是[angular-styleguide](https://github.com/johnpapa/angular-styleguide/blob/master/a1/i18n/zh-CN.md)。
<a name="best-practices"></a>
## 2 最好的代码实践

- [建议]一个文件中应该被限制多少个angular组件，这里的组件指的是`controller`，`directive`等，默认为1。对应规则[component-limit](https://github.com/Gillespie59/eslint-plugin-angular/blob/development/docs/component-limit.md)
- [建议]在定义router或state的时候，应该使用angular的controllerAs语法。对应规则[controller-as-route](https://github.com/Gillespie59/eslint-plugin-angular/blob/development/docs/controller-as-route.md)
	
	```
	// valid
	$routeProvider.when('/myroute', {
		controller: 'MyController',
		controllerAs: 'vm'
	});
	$routeProvider.when('/myroute', {
		controller: 'MyController as vm'
	});
	```
- [建议]使用controllerAs语法时把this 赋值给一个可捕获的变量，选择一个有代表性的名称，例如vm代表ViewModel。对应规则[controller-as-vm](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/controller-as-vm.md)

	> 为什么？：this在不同的地方有不同的语义（就是作用域不同），在controller中的一个函数内部使用this时可能会改变它的上下文。用一个变量来捕获this的上下文从而可以避免遇到这样的坑。

	```
	/* recommended */
	function Customer () {
		var vm = this;
		vm.name = {};
		vm.sendMessage = function() { };
	}
	```
- [建议]不要在`$scope`上面添加属性，使用`controllerAs`语法将数据和属性添加到`this`上。对应规则[controller-as](https://github.com/Gillespie59/eslint-plugin-angular/blob/development/docs/controller-as.md)

	> 为什么？：`controllerAs` 是`$scope`的语法修饰，你仍然可以绑定到View上并且访问`$scope`的方法。
	> 为什么？：避免在`controller`中使用`$scope`，最好不用它们或是把它们移到一个`factory`中。factory中可以考虑使用`$scope`，`controller`中只在需要时候才使用`$scope`，例如当使用`$emit`，`$broadcast`，或者`$on`。

	```
	/* avoid */
	function Customer ($scope) {
		$scope.name = {};
		$scope.sendMessage = function() { };
	}
	/* recommended - but see next section */
	function Customer () {
		this.name = {};
		this.sendMessage = function() { };
	}
	```
- [强制]定义`promise`的时候不要使用`$q.deferred`，而是`$q(function(resolve, reject){})`。对应规则[deferred](https://github.com/Gillespie59/eslint-plugin-angular/blob/development/docs/deferred.md)
- [强制]没有被使用到的依赖不要进行注入。对应规则[di-unused](https://github.com/Gillespie59/eslint-plugin-angular/blob/development/docs/di-unused.md)
- [建议]当创建一个`directive`需要作为一个独立元素时，`restrict`值设置为E（自定义元素），也可以设置可选值`A`（自定义属性）。一般来说，如果它就是为了独立存在，用E是合适的做法。一般原则是允许`EA`，但是当它是独立的时候这更倾向于作为一个元素来实施，当它是为了增强已存在的DOM元素时则更倾向于作为一个属性来实施。定义指令的时候，`restrict`属性只能设置为A、E或AE，不能使用C和M。对应规则[directive-restrict](https://github.com/Gillespie59/eslint-plugin-angular/blob/development/docs/directive-restrict.md)

	> 为什么？：虽然我们允许directive被当作一个class来使用，但如果这个directive的行为确实像一个元素的话，那么把directive当作元素或者属性是更有意义的。
- [强制]不要定义空的controller。对应规则[empty-controller](https://github.com/Gillespie59/eslint-plugin-angular/blob/development/docs/empty-controller.md)

	```
	// invalid
	angular.module('myModule').controller('EmptyController', function () {
	});
	```
- [强制]不允许使用`template`属性定义模块，建议使用`templateUrl`，从外部文件定义模板。对应规则[no-inline-template](https://github.com/Gillespie59/eslint-plugin-angular/blob/development/docs/no-inline-template.md)  
	**注：当模板有不超过两个闭合标签或一个自闭和标签的时候，使用template不会视为错误**
- [强制]不允许注入特定的`service`，如`$http`不建议注入到`controller`中。对应规则[no-services](https://github.com/Gillespie59/eslint-plugin-angular/blob/development/docs/no-services.md)
- [强制]`$scope`对象中的$on和$watch方法应该被赋值到变量中，以便在$destroy事件中删除。对应规则[on-watch](https://github.com/Gillespie59/eslint-plugin-angular/blob/development/docs/on-watch.md)   

	```
	// invalid
	$rootScope.$on('event', function () {
   		// ...
	});
	
	// valid
	$scope.$on('event', function () {
		// ...
	});
	// valid
	var unregister = $rootScope.$on('event', function () {
   		// ...
	});
	```
- [强制]不允许使用以`$$`开头的`angular`私有变量。对应规则[no-private-call](https://github.com/Gillespie59/eslint-plugin-angular/blob/development/docs/no-private-call.md)  

<a name="deprecated-angular-features"></a>
## Angular弃用的特性	

- [强制]在Angular1.4中，`$cookieStore`这个`service`已经被弃用了，取而代之的是`$cookies`。对应规则[no-cookiestore](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/no-cookiestore.md)
- [强制]1.3之后的版本中，定义指令的时候不允许使用`replace`属性。对应规则[no-directive-replace](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/no-directive-replace.md)
- [强制]不允许使用`$http`方法返回的`Promise`的`success`和`error`方法，应该使用标准的`promiseAPI`，即`.then()`。对应规则[no-http-callback](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/no-http-callback.md)

<a name="conventions"></a>
## 约定俗成
- [建议]依赖注入的参数列表应该按照字母表排序。对应规则[di-order](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/di-order.md)

	```
	angular.module('myModule').factory('myService', function($http, $location, $q, myService, someService) {
   		// ...
	});
	```
- [强制]所有的依赖注入使用同一种方法。可选`array`，`function`和`$inject`。对应规则[di](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/di.md)
- [建议]指定回调函数使用命名函数亦或是匿名函数。默认使用命名函数。对应规则[function-type](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/function-type.md)

	```
	angular.module('app',[])
	.controller('DemoController',controllerFn);
	```
- [建议]**模块**的依赖项应该有一个合理的排列顺序。一般按照`Angular标准模块->第三方模块->自定义模块`三个分组进行排序每一个分组内按字母表顺序排列。对应规则[module-dependency-order](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/module-dependency-order.md)

	```
	angular.module('myApp',[
	'ngAnimate','ngCookies','ngSanitize', // 这是ng标准模块
	'ui.bootstrap','ui.router',           // 这是第三方模块
	'myAppModules','myAppRouter'])		  // 这是自定义模块
	```
- [建议]每一个依赖注入独占一行。对应规则[one-dependency-per-line](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/one-dependency-per-line.md)

	```
	app.controller('MyController', [
		'$http',
		'$q',
	function($http,
   		      $q) {
	}]);
	```
- [建议]调用REST API的service中应该使用统一的rest service，可选择的有`'$http'`、`'$resource'`和`'Restangular'`。对应规则[rest-service](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/rest-service.md)
- [建议]手动触发脏数据检查的时候使用`$apply`方法还是`$digest`，两者的不同点在于:`$digest`从我们调用方法的作用域开始执行数据检查，而$apply方法会从$rootScope逐层往下进行数据检查。建议使用`$digest`。对应规则[watchers-execution](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/watchers-execution.md)

<a name="angular-wrappers"></a>
## Angular封装
- [建议]使用`angular.element`作为选择器，而不是使用`$`或`jQuery`。即使代码中使用了`jQuery`，`angular.element`也会调用`jQuery`的方法。对应规则[angularelement](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/angularelement.md)
- [强制]判断一个变量是否是`undefined`的时候使用`angular.isUndefined`或`angular.isDefined`，注意不要使用`!angular.isUndefined(foo)`或`!angular.isDefined(foo)`。对应规则[definedundefined](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/definedundefined.md)

	```
	// invalid
	value === undefined 
	// invalid
	value !== undefined 
	// invalid
	!angular.isUndefined(value) 
	// invalid
	!angular.isDefined(value) 
	```
- [强制]使用`$document`代替`document`。对应规则[document-service](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/document-service.md)
- [强制]使用`$window`代替`window`。对应规则[window-service](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/window-service.md)
- [强制]使用`angular.forEach`方法来进行遍历，不使用数组原生的`Array.prototype.forEach`方法。对应规则[foreach](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/foreach.md)

	```
	// invalid
	someArray.forEach(function (element) {
   		// ...
	});
	// valid
	angular.forEach(someArray, function (element) {
   		// ...
	});
	```
- [强制]使用`$interval`代替`setInterval`。对应规则[interval-service](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/interval-service.md)
- [强制]使用`$timeout`代替`setTimeout`。对应规则[timeout-service](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/timeout-service.md)
- [强制]使用`angular.fromJson`代替`JSON.parse`，使用`angular.toJson`代替`JSON.stringify`。对应规则[json-functions](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/json-functions.md)
- [强制]使用`$log service`代替`console`中的方法，包括`log()`、`warn()`、`error()`、`debug()`和`info()`。对应规则[log](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/log.md)
- [强制]经过`angular.element`初始化的方法已经是`jqlite`对象了，不需要再用`$`或`jQuery`。对应规则[no-jquery-angularelement](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/no-jquery-angularelement.md)

	```
	// invalid
	$(angular.element("#id"))
	```
- [强制]判断一个值是否是数组的时候使用`angular.isArray()`。对应规则[typecheck-array](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/typecheck-array.md)

	```
	// invalid
	Array.isArray(someArray)
	// valid
	angular.isArray(someArray);
	```
- [强制]判断一个值是否是日期对象的时候使用`angular.isDate()`。对应规则[typecheck-date](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/typecheck-date.md)
- [强制]判断一个值是否是函数的时候使用`angular.isFunction()`。对应规则[typecheck-function](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/typecheck-function.md)
- [强制]判断一个值是否是数字的时候使用`angular.isNumber()`。对应规则[typecheck-number](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/typecheck-number.md)
- [强制]判断一个值是否是对象的时候使用`angular.isObject()`。对应规则[typecheck-object](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/typecheck-object.md)
- [强制]判断一个值是否是字符串的时候使用`angular.isString()`。对应规则[typecheck-string](https://github.com/Gillespie59/eslint-plugin-angular/blob/master/docs/typecheck-string.md)

**[⬆ 返回目录](#table-of-contents)**


## Reference
- [eslint-plugin-angular](https://github.com/Gillespie59/eslint-plugin-angular)
- [ESLint-rules-docs-cn](https://github.com/y8n/ESLint-rules-docs-cn/blob/master/plugins/eslint-plugin-angular-rules.md)
- [angular-styleguide zh-CN](https://github.com/johnpapa/angular-styleguide/blob/master/a1/i18n/zh-CN.md)
