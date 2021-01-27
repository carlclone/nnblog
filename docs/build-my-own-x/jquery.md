# 构建自己的 jQuery

## 前言

一直很好奇jQuery 的$("#id") 的内部到底是一个什么东西, 于是有了这次构建之旅


首先说结论, $("#id") 等价于 PHP 中的

```php
// dollar == $
function dollar($selector) {
    return new jQuery($selector);
} 

dollar("#id");

```

也就是说`$("#id")` 其实就是 jQuery 实例的工厂方法, 执行后得到的是一个 jQuery 的实例, 然后会根据 selector 在 DOM 树中找到目标 DOM 元素, 作为 jQuery 实例的一个属性, 作为后续方法调用的实际操作对象, 各种方法其实就是原生 DOM 操作的封装

看到这里本来是可以结束了的, 但我没想到 JS 的面向对象居然如此与众不同



## 作用域链问题

第一个让我困惑的是这个语法

```
var $ = (function () {        // <-- 为啥又包了一层

	'use strict';

	/**
	 * Create the constructor
	 * @param {String} selector The selector to use
	 */
	var Constructor = function (selector) {
		this.elems = document.querySelectorAll(selector);  
	};

	/**
	 * Return the constructor
	 */
	return Constructor;

})();
```

为什么不直接 `var Constructor = function ....` , 要再在外面包一个函数呢? 查找学习之后发现这个东西在 JS 里叫 [revealing module pattern](https://vanillajstoolkit.com/boilerplates/#Revealing-Module-Pattern) , 理解这个东西需要知道 JS 有作用域链这个设计, 当一个变量在当前作用域找不到的时候, 它会一直向上级作用域查找同名变量. 这就造成了如果直接在当前作用域定义临时变量用于生成实例, 很有可能这些临时变量会影响到后面作用域, 造成污染, 使用这么一个函数包起来定义临时变量, 就相当于在作用域链上开了一个分支链, 反正不会有人继续继承这一条链, 也就不会造成污染问题



另外他的名字叫做 module pattern 的原因,我的理解是类似 ES6 的 export 语法, 只暴露这个模块的一部分内容到外界, 使外界不需要关注更多细节, 对应到 PHP 里就是 public / private 的应用



但是这个时候, 实例化 jQuery 需要使用关键字 new , `var sandwiches = new $('.sandwiches');` , 只有使用 new 关键词, JS 才会把 `function (selector) {...`作为构造函数, 返回一个对象, 否则得到的是函数的返回值



为了减少一个 new 的敲键盘消耗, jQuery 做的是把 new XXX 的操作封装到一个函数里



```
var $ = (function () {

	'use strict';

	/**
	 * Create the constructor
	 * @param {String} selector The selector to use
	 */
	var Constructor = function (selector) {
		if (selector === 'document') {
			this.elems = [document];
		} else if (selector === 'window') {
			this.elems = [window];
		} else {
			this.elems = document.querySelectorAll(selector);
		}
	};

	/**
	 * Instantiate a new constructor
	 */
	var instantiate = function (selector) {     // <-- 这里
		return new Constructor(selector);
	};

	/**
	 * Return the constructor instantiation
	 */
	return instantiate;

})();
```



## 原型链问题

在我的理解里, 方法直接定义在 Constructor 里就可以了

```
var Constructor = function (selector) {
		if (selector === 'document') {
			this.elems = [document];
		} else if (selector === 'window') {
			this.elems = [window];
		} else {
			this.elems = document.querySelectorAll(selector);
		}
		
		this.func = function() {console.log("this is func")}
	};
```

但是看到的文章里使用了 prototype 的形式定义, 查资料得知在原型上定义的话,相当于PHP 里"继承"的功能, 并且在内存里只会占用一份空间, 而在 Constructor 里定义的话每个实例都会需要额外的内存来保存这个方法 

定义在函数的原型上的方法就可以顺利访问当前实例的` this.xxx `属性

```
/**
 * Run a callback on each item
 * @param  {Function} callback The callback function to run
 */
Constructor.prototype.each = function (callback) {
	if (!callback || typeof callback !== 'function') return;
};
```

完整代码

```
var $ = (function () {

	'use strict';

	/**
	 * Create the constructor
	 * @param {String} selector The selector to use
	 */
	var Constructor = function (selector) {
		if (!selector) return;
		if (selector === 'document') {
			this.elems = [document];
		} else if (selector === 'window') {
			this.elems = [window];
		} else {
			this.elems = document.querySelectorAll(selector);
		}
	};

	/**
	 * Do ajax stuff
	 * @param  {String} url The URL to get
	 */
	Constructor.prototype.ajax = function (url) {
		// Do some XHR/Fetch thing here
		console.log(url);
	};

	/**
	 * Run a callback on each item
	 * @param  {Function} callback The callback function to run
	 */
	Constructor.prototype.each = function (callback) {
		if (!callback || typeof callback !== 'function') return;
		for (var i = 0; i < this.elems.length; i++) {
			callback(this.elems[i], i);
		}
		return this;
	};

	/**
	 * Add a class to elements
	 * @param {String} className The class name
	 */
	Constructor.prototype.addClass = function (className) {
		this.each(function (item) {
			item.classList.add(className);
		});
		return this;
	};

	/**
	 * Remove a class to elements
	 * @param {String} className The class name
	 */
	Constructor.prototype.removeClass = function (className) {
		this.each(function (item) {
			item.classList.remove(className);
		});
		return this;
	};

	/**
	 * Instantiate a new constructor
	 */
	var instantiate = function (selector) {
		return new Constructor(selector);
	};

	/**
	 * Return the constructor instantiation
	 */
	return instantiate;

})();
```


## 结束

构建的过程其实是一个 JavaScript 面向对象编程的尝试, 整个面向对象的模型和其他语言完全不一样, 由于作用域链,原型链这两个 JS 独有的设计, 也会有些坑容易踩到

## 参考资料

[How to create your own vanilla JS DOM manipulation library like jQuery
](https://gomakethings.com/how-to-create-your-own-vanilla-js-dom-manipulation-library-like-jquery/)

[Methods Within Constructor vs Prototype in Javascript](https://www.thecodeship.com/web-development/methods-within-constructor-vs-prototype-in-javascript/#:~:text=Prototype%20will%20enable%20us%20to,to%20one%20common%20prototype%20object.)