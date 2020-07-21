# JS相关

#### 一、原型/构造函数/实例/原型链

**1. 原型`(prototype)`**

一个简单的对象，用于实现对象的属性继承。可以简单的理解成对象的爹。在 Firefox 和 Chrome 中，每个`JavaScript`对象中都包含一个`__proto__` (非标准)的属性指向它爹(该对象的原型)，可`obj.__proto__`进行访问。

**2. 构造函数**

可以通过`new`来 新建一个对象的函数。

**3. 实例**

通过构造函数和`new`创建出来的对象，便是实例。 实例通过`__proto__`指向原型，通过`constructor`指向构造函数。

```伪代码
实例.__proto__ === 原型
原型.constructor === 构造函数
构造函数.prototype === 原型
```

**4. 原型链**

**原型链是由原型对象组成**，每个对象都有 `__proto__` 属性，指向了创建该对象的构造函数的原型，`__proto__` 将对象连接起来组成了原型链。是一个用来**实现继承和共享属性**的有限的对象链。

- **属性查找机制**: 当查找对象的属性时，如果实例对象自身不存在该属性，则沿着原型链往上一级查找，找到时则输出，不存在时，则继续沿着原型链往上一级查找，直至最顶级的原型对象`Object.prototype`，如还是没找到，则输出`undefined`；
- **属性修改机制**: 只会修改实例对象本身的属性，如果不存在，则进行添加该属性，如果需要修改原型的属性时，则可以用: `b.prototype.x = 2`；但是这样会造成所有继承于该对象的实例的属性发生改变。



#### 二、new 的实现原理

1. 创建一个空对象，构造函数中的this指向这个空对象
2. 这个新对象被执行 [[原型]] 连接
3. 执行构造函数方法，属性和方法被添加到this引用的对象中
4. 如果构造函数中没有返回其它对象，那么返回this，即创建的这个的新对象，否则，返回构造函数中返回的对象。

```javascript
function _new () {
	let target = {}; // 创建一个空对象
  let [constructor, ...args] = [...arguments];
  target.__proto__ = constructor.prototype; // 原型连接
  let result = constructor.apply(target, args); // 执行构造函数，this指向target，添加属性和方法
  if (result && (typeof(result) == 'object' || typeof(result) == 'function')) {
    return result; // 如果构造函数返回的是一个对象，返回这个对象
  }
  return target; // 否则，返回创建的新对象
}
```



#### 三、ES5的继承方式

1. 原型链：将一个类型的实例赋值给另一个构造函数的原型

   `SubType.prototype = new SuperType();`

2. 构造函数继承：在子类型构造函数的内部调用超类型构造函数

   ```javascript
   function SubType() {
     SuperType.call(this);
   }
   ```

[还有4种](https://www.jianshu.com/p/72fea052ed05)



#### 四、执行上下文

执行上下文可以简单理解为一个对象:

- 它包含三个部分:
  - 变量对象(VO)：可以抽象为一种数据作用域，其实也可以理解为就是一个简单的对象，它存储着该执行上下文中的所有变量和函数声明(不包含函数表达式)。
  - 作用域链(词法作用域)：可以理解为一组对象列表，包含父级和自身的变量对象，因此我们便能通过作用域链访问到父级里声明的变量或者函数。
  - `this`指向
- 它的类型:
  - 全局执行上下文
  - 函数执行上下文
  - `eval`执行上下文
- 代码执行过程:
  - 创建 **全局上下文** (global EC)
  - 全局执行上下文 (caller) 逐行 **自上而下** 执行。遇到函数时，**函数执行上下文** (callee) 被`push`到执行栈顶层
  - 函数执行上下文被激活，成为 active EC, 开始执行函数中的代码，caller 被挂起
  - 函数执行完后，callee 被`pop`移除出执行栈，控制权交还全局上下文 (caller)，继续执行



#### 五、Event Loop

Event Loop即事件循环，是指浏览器或Node的一种解决JavaScript单线程运行时不会阻塞的一种机制，也就是我们经常使用**异步**的原理。

> "**Event Loop是一个程序结构，用于等待和发送消息和事件。**（a programming construct that waits for and dispatches events or messages in a program.）"

**事件循环的进程模型**

- 选择当前要执行的任务队列，选择任务队列中最先进入的任务，如果任务队列为空即`null`，则执行跳转到微任务（`MicroTask`）的执行步骤。
- 将事件循环中的任务设置为已选择任务。
- 执行任务。
- 将事件循环中当前运行任务设置为null。
- 将已经运行完成的任务从任务队列中删除。
- microtasks步骤：进入microtask检查点。
- 更新界面渲染。
- 返回第一步。

[参考](https://zhuanlan.zhihu.com/p/55511602)



#### 六、闭包

> 闭包属于一种特殊的作用域，称为 **静态作用域**。它的定义可以理解为: **父函数被销毁** 的情况下，返回出的子函数的`[[scope]]`中仍然保留着父级的单变量对象和作用域链，因此可以继续访问到父级的变量对象，这样的函数称为闭包。

闭包会产生一个很经典的问题:

- 多个子函数的`[[scope]]`都是同时指向父级，是完全共享的。因此当父级的变量对象被修改时，所有子函数都受到影响。

解决:

- 变量可以通过 **函数参数的形式** 传入，避免使用默认的`[[scope]]`向上查找
- 使用`setTimeout`包裹，通过第三个参数传入
- 使用 **块级作用域**，让变量成为自己上下文的属性，避免共享

***闭包的作用***：

1. 能够访问函数定义时所在的词法作用域(阻止其被回收)。

   ```javascript
   function foo() {
     let a = 2;
     return function fn() {
       console.log(a);
     }
   }
   let func = foo();
   func(); // 2
   ```

2. 私有化变量

   ```javascript
   function base() {
     let x = 10; // 私有变量
     return {
       getX: function() {
         return x;
       }
     }
   }
   let obj = base();
   console.log(obj.getX()); // 10
   ```

3. 模拟块级作用域

   ```javascript
   let a = [];
   for (let i = 0; i < 10; i++) {
     a[i] = (function (j) {
   		return function () {
         console.log(j);
       } 
     })(i);
   }
   a[6](); // 6
   ```

4. 创建模块

   ```javascript
   function coolModule() {
     let name = 'Sunny';
     let age = 20;
     function sayName() {
       console.log(name);
     }
     function sayAge() {
       console.log(age);
     }
     return {
       sayName,
       sayAge
     }
   }
   let info = coolModule();
   info.sayName(); // 'Sunny'
   ```

   模块模式具有两个必备的条件(来自《你不知道的JavaScript》)

   > - 必须有外部的封闭函数，该函数必须至少被调用一次(每次调用都会创建一个新的模块实例)
   > - 封闭函数必须返回至少**一个**内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。



#### 七、类型转换

1. -、*、/、% ：一律转换成数值后计算

2. +：

- 数字 + 字符串 = 字符串， 运算顺序是从左到右
- 数字 + 对象， 优先调用对象的`valueOf` -> `toString`
- 数字 + `boolean/null` -> 数字
- 数字 + `undefined` -> `NaN`

3. `[1].toString() === '1'`
4. `{}.toString() === '[object object]'`
5. `NaN!== NaN` 、+undefined 为 NaN



#### 八、[Arguments 对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/arguments)

`arguments`对象是所有（非箭头）函数中都可用的**局部变量**。你可以使用`arguments`对象在函数中引用函数的参数：`arguments[0]`。

`arguments`对象不是一个 [`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Array) 。它类似于`Array`，但除了length属性和索引元素之外没有任何`Array`属性。例如，它没有 [pop](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/pop) 方法。但是它可以被转换为一个真正的`Array`：

如果调用的参数多于正式声明接受的参数，则可以使用`arguments`对象。这种技术对于可以传递可变数量的参数的函数很有用。使用 `arguments.length`来确定传递给函数参数的个数，然后使用`arguments`对象来处理每个参数。要确定函数[签名](https://developer.mozilla.org/zh-CN/docs/Glossary/Signature/Function)中（输入）参数的数量，请使用`Function.length`属性。



#### 九、函数柯里化

在一个函数中，首先填充几个参数，然后再返回一个新的函数的技术，称为函数的柯里化。通常可用于在不侵入函数的前提下，为函数预置通用参数，供多次重复调用。

函数柯里化的主要作用：

- 参数复用
- 提前返回 – 返回接受余下的参数且返回结果的新函数
- 延迟执行 – 返回新函数，等待执行

```js
const curry = (fn, ...args) => {
  return args.length < fn.length 
    ? (...arguments) => curry(fn, ...args, ...arguments)
  	: fn(...args);
}
function sumFn(a, b, c) {
  return a + b + c;
}
let sum = curry(sumFn);
console.log(sum(2)(3)(5)); // 10
console.log(sum(2, 3, 5)); // 10
console.log(sum(2)(3, 5)); // 10
console.log(sum(2, 3)(5)); // 10
```



#### 十、抽象语法树 (AST: Abstract Syntax Tree)

是将代码逐字母解析成 **树状对象** 的形式。这是语言之间的转换、代码语法检查，代码风格检查，代码格式化，代码高亮，代码错误提示，代码自动补全等等的基础。



#### 十一、babel编译原理

- babylon 将 ES6/ES7 代码解析成 AST
- babel-traverse 对 AST 进行遍历转译，得到新的 AST
- 新 AST 通过 babel-generator 转换成 ES5



#### 十二、JSONP的原理

尽管浏览器有同源策略，但是 标签的 `src` 属性不会被同源策略所约束，可以获取任意服务器上的脚本并执行。`jsonp` 通过插入 `script` 标签的方式来实现跨域，参数只能通过 `url` 传入，仅能支持 `get `请求。

jsonp源码实现：

```javascript
function jsonp({url, params, callback}) {
  return new Promise((resolve, reject) => {
    // 创建script标签
    let script = document.createElement('script');
    // 将回调函数挂在window上
    window[callback] = function(data) {
      resolve(data);
      // 代码执行后，删除插入的script标签
      document.body.removeChild(script);
    }
    // 回调函数加在请求地址上
    params = {..params, callback};
    let arrs = [];
    for (let key in params) {
      arrs.push(`${key}=${params[key]}`);
    }
    script.src = `${url}?${arrs.join('&')}`;
    document.body.appendCild(script);
  });
}
```



####十三、JSBridge原理

JavaScript单独运行在一个JS Context中，与原生运行环境天然隔离。我们可以将这种情况与 RPC（Remote Procedure Call，远程过程调用）通信进行类比，将 Native 与 JavaScript 的每次互相调用看做一次 RPC 调用。

在 JSBridge 的设计中，可以把前端看做 RPC 的客户端，把 Native 端看做 RPC 的服务器端，从而 JSBridge 要实现的主要逻辑就出现了：通信调用（Native 与 JS 通信） 和句柄解析调用。

1. **JS 调用 Native**

   1.1 **注入API**：Native通过WebView向JavaScript的context(window)注入对象或方法，让js调用时，Native执行相应的逻辑。

   ```javascript
   window.webkit.messageHandlers.openPage.postMessage({ url: url }); // iOS
   window.appSdk.openPage(url); // Android
   ```

   1.2 **拦截URL SCHEME**：Web 端通过某种方式（例如 iframe.src）发送 URL Scheme 请求，之后 Native 拦截到请求并根据 URL SCHEME（包括所带的参数）进行相关操作。

   > 为什么选择 iframe.src 不选择 locaiton.href ？
   >
   > 因为如果通过 location.href 连续调用 Native，很容易丢失一些调用。

   *创建请求，需要一定的耗时，比注入 API 的方式调用同样的功能，耗时会较长。所以推荐使用注入API的方式。*

2. **Native 调用 JS**

   web定义在window上的全局方法，native直接调用。



#### 十四、求字符长度

charCodeAt(n):以Unicode编码返回指定位置索引。由于中文字符Unicode编码大于255，故而能够得出一个字符串的实际长度。

```javascript
function strLength(str) {
	if(str.length == 0 || !str) return null;
	var l = str.length;
	for(var i = 0; i< str.length; i++){
		if(str.charCodeAt(i) > 255) l++;
	}
	return l;
}
```



