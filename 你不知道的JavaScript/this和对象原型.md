## this和对象原型

### this

this是在运行时绑定的，而不是在编写时绑定的。this的绑定和函数声明的位置没有任何区别，只取决于函数的调用方式。

#### 默认绑定

最常见的是独立函数调用

```js
function foo(){
	console.log(this.a);
}

var a = 2;
foo();  //2
```

这里this绑定到了全局对象。值得注意的是，只有在非严格模式下才会有这样的结果，严格模式下会绑定到undefined。

#### 隐式绑定

```js
function foo(){
	console.log(this.a);
}

var obj = {
	a: 2,
	foo: foo
}

obj.foo();  //2
```

函数调用的位置如果是被某个对象拥有或包含，那么this会指向这个对象。

##### 隐式丢失

```js
function foo(){
	console.log(this.a);
}

var obj = {
	a:2,
	foo:foo
}

var bar = obj.foo;

var a = "opps,global";

bar();  //"opps,global"
```

bar引用的是foo函数本身，因此这里应用的是默认绑定。

值得注意的是，参数传递或者赋值引用都可能导致this绑定丢失。

#### 显式绑定

即使用call方法和apply方法

```js
function foo(){
	console.log(this.a);
}

var obj = {
	a:2
};

foo.call(obj);  //2
```

通过foo.call我们可以在调用foo时强制把它的this绑定到obj上。

如果传入一个字符串，布尔值或者数字，这个原始值会被转换为它的对象形式，这个过程叫做装箱。

#### new绑定

```js
function foo(a){
	this.a = a;
}

var bar = new foo(2);
console.log(bar.a);  //2
```

这里foo的this绑定到了新创建的对象上。

#### 判断this

1. 函数是否在new中调用，如果是，this绑定的是新创建的对象
2. 函数是否通过call,apply调用，如果是，绑定的是指定的对象
3. 函数是否在上下文中被调用，如果是，绑定的是上下文对象
4. 如果都不是，则视为默认绑定

#### 绑定例外

将null/undefined传入call/apply/bind时，会被忽略，实际应用的可能是默认绑定。



