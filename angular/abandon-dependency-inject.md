# 结合 ES6 消除 Angular1.5 中的 DI(依赖注入)

大部分内容在 [Angular1.x + ES6 开发风格指南](https://github.com/kuitos/kuitos.github.io/issues/34) 中均有说明,这里再做一个整理&补充


## 业务系统不需要定义自己的 Service/Filter, 全部用 ES6 Module 代替

## 不直接使用angular内置的服务,系统自己做上层包装
    
$http/$resource: [rs-generator]()

## 必须使用的时候,通过 injector get 的方式
    
```js
import injector from 'angular-es-utils/injector';
    
class Controller {
    
	getName() {
		const $timeout = injector.get('$timeout');
			$timeout(() => {}, 100);
		}
	}
}	
```


