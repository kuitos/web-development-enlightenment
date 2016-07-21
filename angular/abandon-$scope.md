# 结合ES6消除Angular1.5 中的$scope

大部分内容在 [Angular1.x + ES6 开发风格指南](https://github.com/kuitos/kuitos.github.io/issues/34) 中均有说明,这里再做一个整理&补充

## Step 1. 使用 controllerAs 语法

* 路由
	
	bad
	
	```js
	function Controller($scope) {
		...
	}
	Controller.$inject = ['$scope'];
	angular.controller('');
	...
	$stateProvider.state('app', {
		url: '/',
		controller: 'Controller'
	});
	```
	good
	
	```js
	export default Controller {
		
		contructor() {
			this.name = 'kuitos';
		}
		
		getName() {
			return this.name;
		}
		
	}
	
	...
	import Controller from './Controller';
	
	$stateProvider.state('app', {
		url: '/',
		controller: Controller,
		controllerAs: '$ctrl'
	});

	```
	
* 页面划分的控制器区块
	
	bad (ng-controller方式)
	
	```js
	import Controller from './Controller';
	...
	angular.controller('Controller', Controller);
	```
	
	```html
	<div ng-controller="Controller as ctrl">
		...
	</div>
	...
	<div ng-controller="Controller1 as ctrl1">
		...
	</div>
	```
	
	good (组件方式)
	
	```js
	import controller from './Controller';
	import template from './template.html';
	
	const ddo = {
		template,
		controller,
		bindings: {
			businessId: '<'
		}
	};
	angular.component('xxContainerA', ddo);
	```
	
	```html
	<xx-container-a business-id="$ctrl.idA"></container-a>
	...
	<xx-container-b business-id="$ctrl.idB"></container-b>
	```
	
## Step 2. 依赖属性计算

bad (`$scope.$watch`)	

```js
class Controller {
	
	constructor($scope) {
		$scope.$watch('name', newVal => {
			console.log(newVal);
		});
	}
}
```

good (getter/setter)

```js
class Contructor() {

	_onNameChange() {
		console.log(`name changed to ${this.name}`);
	}

	get name() {
		return this._name;
	}
	
	set name(value) {
		if(value !== this._name) {
			this._name = value;			
		}
	}
}
```

good (component $onChanges)

```js
angular.component('component', {
	template,
	controller,
	bindings: {
		name: '<'
	}
});
....

class ComponentCtrl {

	$onChanges(changes) {
		
		if(changes.name) {
			this._onNameChange(changes.name.currentValue);
		}
	}
}

```
## Step 3. 事件
原则上是尽量减少事件通知的方式，组件间采用`props + inline-event`的方式做信息传递。例如：

```js
angular.component('component', {
	template,
	controller,
	bindings: {
		name: '<',
		onUpdate: '&?'
	}
});
```

```html
<component name="$ctrl.name" onUpdate="$ctrl.onUpdate(props)"></component>
```

对于无法采用这种方式的 edge case，比如在DOM层级上几乎没有关系的两个组件，这时候只能采用事件机制。但是也不要用 angular $emit/$broadcast/$on api，为了消除angular的影子，可以使用自己实现的EventBus，比如直接用这个： [EventBus](https://github.com/kuitos/angular-es-utils/blob/master/src/event-bus/index.js)