## 加载与执行
### 脚本位置
`<script>`标签可以放在HTML文档的head或body的位置。理论上，把样式和行为有关的脚本放在一起并优先加载，有助于确保页面渲染和交互的正确性。

但是将script标签放在head中存在严重的性能问题：在head中加载标签会导致页面阻塞直到脚本下载执行完成。

把脚本放在页面顶部会导致明显的延迟，通常表现为空白页面，用户无法浏览内容，也无法完成交互。

因此推荐将所有的script标签尽可能放在body标签的底部，以尽可能减少对整个页面下载的影响。

### 组织脚本

每个script标签下载都会导致页面阻塞，而HTTP请求还会带来额外的性能开销，因此下载单个100KB的文件比下载4个25KB的文件更快。也就是说，减少页面中外链脚本的数量可以改善性能。

### 无阻塞脚本

#### 延迟脚本 Deferred Script

script标签拥有defer属性。defer表明这个脚本不会修改DOM，因此可以安全地延迟执行。

```js
<script type="text/javascript" src="file.js" defer></script>
```

带有defer属性的script标签可以放在文档的任何位置，这个script标签将在被解析到时开始下载，但是并不会被执行，直到DOM加载完成（onload事件触发前）。当一个带有defer属性的js文件开始下载时，它不会阻塞浏览器的其他进程，可以与页面中的其他资源一起并行下载。

#### 动态脚本 Dynamic Script

使用标准的DOM方法可以很容易得创建一个`script`元素。

```js
var script = document.createElement("script");
script.type = "text/javascript";
script.src = "file.js"
document.getElementsByTagName("head")[0].appendChild(script);
```

这个新创建的script元素加载了file.js，这个文件在该元素被添加到页面时开始下载而且它的下载与执行不会阻塞页面加载其他进程。

#### XHR脚本注入 XHR Script Injection

先创建一个XHR对象，用它下载js文件，最后通过创建动态script的方式注入到页面中。

这种方法的优点是下载的js代码不会被立刻执行，你可以把脚本的执行推迟到准备好的时候。

## 数据存取

计算机科学中有一个经典的问题是通过改变数据的存储位置来获取最佳的读写性能。

这个问题在js中相对简单，因为只有几种存储方案可以被选择。js有下面四种基本的数据存取位置:

字面量： 字面量仅代表自身，不存在特定的位置，包括 字符串 、数字、布尔值、对象、数组、函数、正则表达式、null和undefined

本地变量：使用var声明定义的数据存储单元

数组元素：存储在js数组对象内部，以数字为索引

对象成员：存储在js对象内部，以字符串为索引

每一种数据存储的位置都有不同的读写消耗，大多数情况下，从一个字面量和一个局部变量中存取数据的性能差异是微不足道的。而访问数组元素和对象成员的代价则更高。通常的建议是，如果在乎运行速度，尽量减少数组项和对象成员的使用。不过也有几种模式来优化代码。

### 管理作用域

#### 作用域链 

在全局环境下定义一个函数，又在函数的执行环境下再定义其他变量，这样就形成了一个作用域链。新创建的往往被称为活动对象并被推到作用域链首位，而作用域链的末尾往往就是全局变量了。在执行过程中，每遇到一个标识符，都会从作用域链头部开始搜索。（如果存在两个同名变量，那么作用域链前面的会遮蔽后面的）。

#### 标识符号解析的性能

一个标识符所在的位置越深，它的读写速度往往就越慢。（目前更新过的主流浏览器经过优化后并不存在这个问题）。

一个建议是尽可能使用局部变量，如果一个跨作用域的值被频繁访问，那么就先把它存储为局部变量。

#### 改变作用域链

一般来说作用域链是不会改变的，有两个语句可以临时改变作用域。

第一个是with。当代码执行到with时，执行环境的作用域链会被临时改变。一个新的变量对象会被创建，这个对象会被推到作用域链的首位。例如如果使用`with(document)`这样的语句，那么`document`下的所有属性都会被提升到作用域首位，而其他变量的位置则会下降。那么访问`document.body`或者`document.getElementById`的速度会变快，而其他对象速度则变慢。

另一个是try-catch语句，在try代码块中发生错误时，执行过程会跳转到catch中去，异常对象在这时会位于作用域首位，其他变量位置下降。

## DOM编程

用脚本操作DOM的代价非常昂贵，它是web应用中最常见的性能瓶颈。

### 浏览器中的DOM

浏览器中通常会把DOM和JavaScript独立实现。这意味着使用js操作DOM天生就慢，就像从一座岛到另一座岛上去需要交付过桥费用一样。推荐的做法是尽可能得待在js的岛上，减少过桥的次数。

### DOM的访问与修改

#### 关于innerHTML方法

直接使用innerHTML要比手动操作DOM更快，但在WebKit浏览器中innerHTML则相对较慢。

（原文充满了时代气息，而innerHTML依旧不被推荐使用）

#### 节点克隆

在大多数浏览器中，节点克隆都要比创建新节点更有效率，尝试使用`element.cloneNode()`替代`document.createElement`。

#### 选择器API

相较于`getElementById`或者`getElementsByTagName`，`querySelectorAll`等选择器api拥有更快的速度（2~6倍）

### 重绘与重排

重绘：Repaints

重排：Reflows

浏览器下载玩HTML，js脚本，css以及图片等资源后，会解析生成两个内部结构：DOM树和渲染树。

DOM树中每一个节点在渲染树中至少存在一个对应节点（隐藏的则没有）。渲染树中的节点称为“帧”或“盒”。一旦DOM树和渲染树都构建完成，浏览器就会绘制（paint）页面元素。如果DOM的变化影响了元素的几何属性，其他元素的位置和几何属性受到影响，浏览器会重新构建渲染树，这个过程称为重排（reflow）。重排后重新绘制到屏幕上称为重绘（repaint）。但如果只是改变了背景色这样的操作，则只会发生重绘，不会有重排，因为元素的布局没有发生变化。

#### 重排的发生时间

+ 添加删除DOM元素
+ 元素位置改变
+ 元素尺寸改变
+ 内容改变（例如一张图片换成了另一张不同大小的图片）
+ 页面渲染器初始化
+ 浏览器窗口尺寸改变

#### 渲染树变化的排队与刷新

一些返回布局信息的方法会触发重排，例如

+ offsetTop,offsetLeft,offsetWidth,offsetHeight
+ scrollTop,scrollLeft,scrollWidth,scrollHeight
+ clientTop,clientLeft,clientWidth,clientHeight
+ getComputedStyle()

尽可能避免使用以上属性，它们都会刷新队列，触发重排以返回最新的值。

#### 最小化重绘和重排

一种比较实用的方法是通过一次性修改cssText来避免反复触发重排，也可以通过修改CSS的class名称而并非修改具体的样式来提高可维护性。

#### 批量修改DOM

当需要对DOM元素进行一系列操作时，以下步骤可以减少重绘和重排

1.使元素脱离文档流

2.对其应用多重改变

3.把元素带回文档中

脱离文档流只需要将display设置为none就可以了，然后就可以进行任意更改而不需要担心触发多次重排。

另一种有效的方法是创建文档片段，使用`document.createDocumentFragment`来避免直接反复更改文档。

#### 缓存布局信息

前文已经阐述了反复获取布局信息会导致重排，而推荐的解决方案是在最开始获取一次布局信息并赋值给一个变量，之后再发生DOM的改变（例如动画）时，不再查询布局信息而直接使用存储好的初始值进行计算。

#### 事件委托

事件绑定会占用大量的处理时间，同时浏览器跟踪每个事件处理器也会占用内存。一个简单而优雅处理DOM事件的技术是事件委托。

事件委托基于事件的冒泡，一个事件发生时会逐步向父元素冒泡直到被捕获。如果我们需要对大量子元素添加事件，那么直接对父元素添加事件监听并且判断事件源就可以直接满足需求。

## 算法与流程控制

### 循环

js中存在四种循环类型，分别是标准for循环、while循环、do-while循环和for-in循环。

在js提供的四种循环类型中，只有for-in循环比其他几种明显要慢。因为在for-in中，每次迭代操作会同时搜索实例和原型属性，会产生额外的开销。

对比相同迭代次数的循环，for-in的速度只有其他的1/7。因此，除非你明确需要迭代一个属性数量未知的对象，否则应该避免使用for-in循环。

#### 减少迭代的工作量

如果每一次循环都需要花费大量事件，那么多次循环意味着更多的事件，限制循环中耗时操作的数量可以提高循环速度。

 比较通用的方法有以下几种：

+ 减少对象成员及数组项的查找次数。
+ 倒序循环可以略微提高性能。

#### 减少迭代次数

减少迭代次数可以获得更加显著的性能提升，最广为人知的限制循环迭代次数的模式被称为“达夫设备”。

一个典型的“Duff's Device”如下：

```js
var iterations = Math.floor(items.length/8),
       startAt = items.length % 8,
       i = 0;
do {
      swich(startAt){
          case 0: process(item[i++]);
          case 7: process(item[i++]);
          case 6: process(item[i++]);
          case 5: process(item[i++]);
          case 4: process(item[i++]);
          case 3: process(item[i++]);
          case 2: process(item[i++]);
          case 1: process(item[i++]);
      }
      startAt = 0;
}while(--iteration);
```

不难看出，达夫设备使得一次迭代中实际执行了多次迭代的操作。

此算法还有一个稍快的版本，取消了switch语句，并将余数处理与主循环分开：

```js
var i = items.length % 8;
while(i){
	process(items[i--]);
}

i = Math.floor(items.length / 8);

while(i){
	process(items[i--]);
	process(items[i--]);
	process(items[i--]);
	process(items[i--]);
	process(items[i--]);
	process(items[i--]);
	process(items[i--]);
	process(items[i--]);
}
```

是否应该使用达夫设备取决于循环次数。如果循环次数小于1000，那么它与常规循环结构只有微不足道的提升。但是如果循环次数超过1000，那么达夫设备的执行效率将明显提升。例如在500000次迭代中，其运行时间比常规循环少70%。

### 基于函数的迭代

原生数组方法forEach()，这个方法遍历一个数组的所有成员，并在每个成员执行一个函数。

```js
items.forEach(function(value,index,array){
	process(value);
});
```

因为数组调用外部方法所会带来额外的开销，所以在所有情况下，基于循环的迭代比基于函数的迭代快8倍。在对运行速度要求严格时，基于函数的迭代并不是合适的选择。

### 条件语句

#### if-else与swich

关于使用if-else还是switch，最流行的方法是基于测试条件的数量来判断。条件数量越大，越倾向于使用switch。

事实证明，大多数情况下switch比if-else运行的要快，但只有当条件数量很大时才快得明显。这两个语句最主要的区别是：当条件增加时，if-else性能负担增加的程度比switch要多。

#### 优化if-else

优化if-else的目标是：最小化到达正确分支前所需判断的条件数量。

+ 最简单的优化方法是确保最可能出现的条件放在首位。
+ 另一种是将if-else组织成嵌套的if-else语句。

#### 查找表

有时候优化条件语句的最佳方案是避免使用if-else和swich。当有大量离散值需要测试时，if-else和swich比使用查找表慢很多。js中可以使用数组和普通对象来构建查找表，查找表访问速度会更快，尤其是在条件语句数量很大的时候。

```js
switch(value){
	case 0:
		return result0;
	case 1:
		return result1;
	case ２:
		return result2;
	default:
		return result3;
}
```

以上转换为查找表的形式：

```js
var results = [result0,result1,result2,result3];
return results[value]
```

### 递归

递归可以把复杂的算法变简单，递归函数的潜在问题是终止条件不明确或者缺少中止条件导致函数长时间运行，并使得用户见面处于假死状态。而且，递归函数还可能遇到浏览器的”调用栈大小限制“。

#### 调用栈大小限制

js引擎所支持的递归数量与js调用栈大小直接相关。（只有ie例外，它的调用栈与系统空闲有关）

不同浏览器关于调用栈溢出错误会反映出不同的异常类型。

#### 递归模式

当遇到调用栈大小限制的问题是，应该考虑到两种不同的递归模式。

* 直接递归模式，即函数调用自身。

  ```js
  function recurse(){
  	recurse();
  }
  recurse();
  ```

+ 隐伏模式，两个函数相互调用形成循环。

  ```js
  function first(){
  	second();
  }
  function second(){
  	first();
  }
  first();
  ```

  

### 迭代

任何递归能实现的算法迭代也可以实现。迭代算法通常包含几个不同的循环，分别对应计算过程的不同方面。

例如合并排序是常见的递归实现的算法：

```js
function merge(left,right){
	var result = [];
	while(left.length > 0 && right.length > 0){
		if(left[0] < right[0]){
			result.push(left.shift());
		}else{
			result.push(right.shift());
		}
	}
	
	return result.concat(left).concat(right);
}

function mergeSort(items){
	if(items.length == 1){
		return items;
	}
	
	var middle = Math.floor(items.length/2),
			left = items.slice(0,middle),
			right = items.slice(middle);
			
	return merge(mergeSort(left),mergeSort(right));
}
```

在这里，mergeSort()会导致很频繁的自调用，一个长为n的数组最终会调用mergeSort()2*n-1次，意味着很可能发生栈溢出错误。

当发生栈溢出错误时并不一定要修改整个算法，这只是表明递归并不是最好的实现方式。

这个合并排序算法同样可以用迭代实现，比如：

```js
function mergeSort(items){
	if(items.length == 1){
		return items;
	}
	
	var work = [];
	for(var i=0,len=items.length;i < len;i++){
		work.push([items[i]]);
	}
	work.push([]); //如果数组长度为奇数
	for(var lim=len;lim > 1;lim = (lim+1)/2){
		for(var j=0,k=0; k < lim;j++,k+=2){
			work[j] = merge(work[k].work[k+1]);
		}
		work[j] = []; //如果数组长度为奇数
	}
	return work[0];
}
```

尽管迭代版本的合并排序算法比递归版本的慢，但它不会像递归版本那样受到调用栈限制的影响。

### Ｍemoization

Memoization是一种避免重复工作的方法，它缓存前一个的计算结果供后续计算使用，避免了重复工作。

Memoization的关键在于创建缓存对象，在计算前检查缓存对象中是否有自己需要的结果。一个简单的例子就是在连续阶乘运算中，我们可以利用缓存对象依次存储0、1、2的阶乘结果，在计算一个阶乘前先检查是否可以使用缓存对象中的值，计算完成后把计算结果也存入缓存对象中。

