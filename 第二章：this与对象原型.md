# this 与 对象原型



## this 是什么

`this` 不是编写时绑定，而是运行时绑定。它依赖于函数调用的上下文条件。`this` 绑定与函数声明的位置没有任何关系，而与函数被调用的方式紧密相连。

当一个函数被调用时，会建立一个称为执行环境的活动记录。这个记录包含函数是从何处（调用栈 —— call-stack）被调用的，函数是 *如何* 被调用的，被传递了什么参数等信息。这个记录的属性之一，就是在函数执行期间将被使用的 `this` 引用。

> 每当你感觉自己正在试图使用 `this` 来进行词法作用域的查询时，提醒你自己：*这里没有桥*。

```js
function foo() {
	var a = 2;
	this.bar();
}

function bar() {
	console.log( this.a );
}

foo(); //undefined
```

### 判定 this

> 与这四种绑定规则不同，ES6 的箭头方法使用词法作用域来决定 `this` 绑定，这意味着它们采用封闭他们的函数调用作为 `this` 绑定（无论它是什么）。它们实质上是 ES6 之前的 `self = this` 代码的语法替代品。

1. 函数是通过 `new` 被调用的吗（**new 绑定**）？如果是，`this` 就是新构建的对象。
   `var bar = new foo()`
2. 函数是通过 `call` 或 `apply` 被调用（**明确绑定**），甚至是隐藏在 `bind` (**硬绑定**) 之中吗？如果是，`this` 就是那个被明确指定的对象。
   `var bar = foo.call( obj2 )`
3. 函数是通过环境对象（也称为拥有者或容器对象）被调用的吗（**隐含绑定**）？如果是，`this` 就是那个环境对象。
   `var bar = obj1.foo()`
4. 否则，使用默认的 `this`（**默认绑定**）。如果在 `strict mode` 下，就是 `undefined`，否则是 `global` 对象。
   `var bar = foo()`

### new

1. 一个全新的对象会凭空创建（就是被构建）

2. *这个新构建的对象会被接入原形链（`[[Prototype]]`-linked）*

3. 这个新构建的对象被设置为函数调用的 `this` 绑定

4. 除非函数返回一个它自己的其他 **对象**，否则这个被 `new` 调用的函数将 *自动* 返回这个新构建的对象。

### 软绑定

```js
if (!Function.prototype.softBind) {
	Function.prototype.softBind = function(obj) {
		var fn = this,
			curried = [].slice.call( arguments, 1 ),
			bound = function bound() {
				return fn.apply(
					(!this ||
						(typeof window !== "undefined" &&
							this === window) ||
						(typeof global !== "undefined" &&
							this === global)
					) ? obj : this,
					curried.concat.apply( curried, arguments )
				);
			};
		bound.prototype = Object.create( fn.prototype );
		return bound;
	};
}
```

```js
function foo() {
   console.log("name: " + this.name);
}

var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };

var fooOBJ = foo.softBind( obj );

fooOBJ(); // name: obj

obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2   <---- 看!!!

fooOBJ.call( obj3 ); // name: obj3   <---- 看!

setTimeout( obj2.foo, 10 ); // name: obj   <---- 退回到软绑定
```



## 对象

### 类型 

JS 的六种主要类型（在语言规范中称为“语言类型”）中的一种：

- `string`

- `number`

- `boolean`

- `null`

- `undefined`

- `object`

注意 *简单基本类型* （`string`、`number`、`boolean`、`null`、和 `undefined`）自身 **不是** `object`。`null` 有时会被当成一个对象类型，但是这种误解源自于一个语言中的 Bug，它使得 `typeof null` 错误地（而且令人困惑地）返回字符串 `"object"`。实际上，`null` 是它自己的基本类型。

`function` 是对象的一种子类型，数组也是一种形式的对象。

#### 内建对象

- `String`

- `Number`

- `Boolean`

- `Object`

- `Function`

- `Array`

- `Date`

- `RegExp`

- `Error`

基本类型值 `"I am a string"` 不是一个对象，它是一个不可变的基本字面值。为了对它进行操作，比如检查它的长度，访问它的各个独立字符内容等等，都需要一个 `String` 对象。

在必要的时候语言会自动地将 `"string"` 基本类型强制转换为 `String` 对象类型，这意味着几乎从不需要明确地创建此类对象。 **强烈推荐** 尽可能地使用字面形式的值，而非使用构造的对象形式。

`null` 和 `undefined` 没有对象包装的形式，仅有它们的基本类型值。相比之下，`Date` 的值 *仅可以* 由它们的构造对象形式创建，因为它们没有对应的字面形式。



### 内容

#### 计算属性名

```js
var prefix = "foo";

var myObject = {
  [prefix + "bar"]: "hello",
  [prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

```js
var myObject = {
  [Symbol.Something]: "hello world"
};
```



#### 属性符描述

- 可写性（Writable）
- 可配置性（Configurable）
- 可枚举性（Enumerable）



#### 存在性

> `in` 和 `hasOwnProperty(..)` 区别于前者查询 `[[Prototype]]` 链，而 `Object.keys(..)` 和 `Object.getOwnPropertyNames(..)` 都 *只* 考察直接给定的对象。

**枚举**

`Object.keys(..)` 返回一个所有可枚举属性的数组，而 `Object.getOwnPropertyNames(..)` 返回一个 *所有* 属性的数组，不论能不能枚举。



### 迭代

`for..in` 循环迭代一个对象上（包括它的 `[[Prototype]]` 链，如果是数组返回下标）所有的可迭代属性。

`for..of` 循环语法，用来迭代数组（和对象，如果这个对象有定义的迭代器），直接迭代值，而不是数组下标（或对象属性）。

```js
var myObject = {
	a: 2,
	b: 3
};

Object.defineProperty( myObject, Symbol.iterator, {
	enumerable: false,
	writable: false,
	configurable: true,
	value: function() {
		var o = this;
		var idx = 0;
		var ks = Object.keys( o );
		return {
			next: function() {
				return {
					value: o[ks[idx++]],
					done: (idx > ks.length)
				};
			}
		};
	}
} );

// 手动迭代 `myObject`
var it = myObject[Symbol.iterator]();
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { value:undefined, done:true }

// 用 `for..of` 迭代 `myObject`
for (var v of myObject) {
	console.log( v );
}
// 2
// 3
```

