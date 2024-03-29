# 深入解读es6特性(deep-into-es6features)

ECMAScript 2015是一项ECMAScript标准，于2015年6月获得批准。

ES2015是该语言的重要更新，也是自2009年ES5标准化以来该语言的第一次重大更新。现在正在主要JavaScript引擎中实现这些功能。

有关 ECMAScript 2015语言的完整规范，请参阅ES2015标准。以下简要介绍仅供参考。

## 箭头函数和this
箭头函数是使用=>语法的函数简写。它们在语法上类似于C＃，Java 8和CoffeeScript中的相关功能。它们支持表达式和语句体。与函数不同，箭头函数与this周围的代码拥有相同的作用域。如果箭头函数在另一个函数内，它共享其父函数的“arguments”变量。

```javascript
// Expression bodies
var odds = evens.map(v => v + 1);
var nums = evens.map((v, i) => v + i);

// Statement bodies
nums.forEach(v => {
  if (v % 5 === 0)
    fives.push(v);
});

// Lexical this
var bob = {
  _name: "Bob",
  _friends: [],
  printFriends() {
    this._friends.forEach(f =>
      console.log(this._name + " knows " + f));
  }
};

// Lexical arguments
function square() {
  let example = () => {
    let numbers = [];
    for (let number of arguments) {
      numbers.push(number * number);
    }

    return numbers;
  };

  return example();
}

square(2, 4, 7.5, 8, 11.5, 21); // returns: [4, 16, 56.25, 64, 132.25, 441]
``` 

## 类
ES2015类比基于原型的OO模式简单。拥有一个方便的声明形式使类模式更易于使用，并鼓励互操作性。类支持基于原型的继承，super调用，实例和静态方法以及构造函数。

```javascript
class SkinnedMesh extends THREE.Mesh {
  constructor(geometry, materials) {
    super(geometry, materials);

    this.idMatrix = SkinnedMesh.defaultMatrix();
    this.bones = [];
    this.boneMatrices = [];
    //...
  }
  update(camera) {
    //...
    super.update();
  }
  static defaultMatrix() {
    return new THREE.Matrix4();
  }
}
``` 

## 增强的对象字面量
扩展了对象字面量，以支持在构造时设置原型，为foo: foo分配提供简写，定义方法和进行super调用。它们一起使对象字面量和类声明更加紧密，让基于对象的设计更方便。

``` javascript
var obj = {
    // Sets the prototype. "__proto__" or '__proto__' would also work.
    __proto__: theProtoObj,
    // Computed property name does not set prototype or trigger early error for
    // duplicate __proto__ properties.
    ['__proto__']: somethingElse,
    // Shorthand for ‘handler: handler’
    handler,
    // Methods
    toString() {
     // Super calls
     return "d " + super.toString();
    },
    // Computed (dynamic) property names
    [ "prop_" + (() => 42)() ]: 42
};
``` 

该__proto__属性需要原生支持，并且在之前的ECMAScript版本中已弃用。大多数引擎现在支持该属性，但有些则不支持。

## 模板字符串
模板字符串为构造字符串提供语法糖。这类似于Perl，Python等中的字符串插值功能。可选的，可以添加标签以允许定制字符串构造，避免注入攻击或从字符串内容构造更高级别的数据结构。

```javascript
// Basic literal string creation
`This is a pretty little template string.`

// Multiline strings
`In ES5 this is
 not legal.`

// Interpolate variable bindings
var name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`

// Unescaped template strings
String.raw`In ES5 "\n" is a line-feed.`

// Construct an HTTP request prefix is used to interpret the replacements and construction
GET`http://foo.org/bar?a=${a}&b=${b}
    Content-Type: application/json
    X-Credentials: ${credentials}
    { "foo": ${foo},
      "bar": ${bar}}`(myOnReadyStateChangeHandler);
``` 

## 解构
解构允许使用模式匹配进行绑定，并支持匹配数组和对象。类似于标准对象查找foo["bar"]，在未找到时产生值undefined。

```javascript
// list matching
var [a, ,b] = [1,2,3];
a === 1;
b === 3;

// object matching
var { op: a, lhs: { op: b }, rhs: c }
       = getASTNode()

// object matching shorthand
// binds `op`, `lhs` and `rhs` in scope
var {op, lhs, rhs} = getASTNode()

// Can be used in parameter position
function g({name: x}) {
  console.log(x);
}
g({name: 5})

// Fail-soft destructuring
var [a] = [];
a === undefined;

// Fail-soft destructuring with defaults
var [a = 1] = [];
a === 1;

// Destructuring + defaults arguments
function r({x, y, w = 10, h = 10}) {
  return x + y + w + h;
}
r({x:1, y:2}) === 23
``` 


## 默认参数+对象展开符+对象收归符
被调用者的默认参数值。在函数调用中将数组转换为连续的参数。将跟随参数绑定到数组。Rest arguments更直接地满足了对常见情况的需求。

``` javascript
function f(x, y=12) {
  // y is 12 if not passed (or passed as undefined)
  return x + y;
}
f(3) == 15


function f(x, ...y) {
  // y is an Array
  return x * y.length;
}
f(3, "hello", true) == 6


function f(x, y, z) {
  return x + y + z;
}
// Pass each elem of array as argument
f(...[1,2,3]) == 6
``` 

## 块级作用域let和const
块范围的绑定构造。let是新的var，const是单一任务。静态限制在分配之前阻止使用。

```javascript
function f() {
  {
    let x;
    {
      // this is ok since it's a block scoped name
      const x = "sneaky";
      // error, was just defined with `const` above
      x = "foo";
    }
    // this is ok since it was declared with `let`
    x = "bar";
    // error, already declared above in this block
    let x = "inner";
  }
}
``` 

## 迭代器+ For..Of
Iterator对象支持自定义迭代，如CLR IEnumerable或Java Iterable。使用通用for..in到基于自定义迭代器的迭代 for..of。不需要实现数组，启用LINQ等惰性设计模式。

```javascript
let fibonacci = {
  [Symbol.iterator]() {
    let pre = 0, cur = 1;
    return {
      next() {
        [pre, cur] = [cur, pre + cur];
        return { done: false, value: cur }
      }
    }
  }
}

for (var n of fibonacci) {
  // truncate the sequence at 1000
  if (n > 1000)
    break;
  console.log(n);
}
```

迭代基于这些duck-typed(鸭子类型)接口（仅使用 TypeScript类型语法进行展示）：

```javascript
interface IteratorResult {
  done: boolean;
  value: any;
}
interface Iterator {
  next(): IteratorResult;
}
interface Iterable {
  [Symbol.iterator](): Iterator
}
``` 

## 生成器Generators
生成器使用`function*`和简化迭代器`yield`。声明为`function *`的函数返回Generator实例。生成器是迭代器的子类型，包括额外的next和throw。这些使得值能够流回到生成器中，因此yield表达式形式返回一个值（或抛出）。

注意：也可用于启用'await'式异步编程，另请参阅ES7 await 提议。

```javascript
var fibonacci = {
  [Symbol.iterator]: function*() {
    var pre = 0, cur = 1;
    for (;;) {
      var temp = pre;
      pre = cur;
      cur += temp;
      yield cur;
    }
  }
}

for (var n of fibonacci) {
  // truncate the sequence at 1000
  if (n > 1000)
    break;
  console.log(n);
}
```

生成器接口是（仅使用TypeScript类型语法进行展示）：

```javascript 
interface Generator extends Iterator {
    next(value?: any): IteratorResult;
    throw(exception: any);
}
```

## Unicode支持
支持完整Unicode的非破坏性添加，包括字符串中的新unicode文字形式和u处理代码点的新RegExp模式，以及用于处理21位代码点级别的字符串的新API。这些新增功能支持在JavaScript中构建全局应用程序。

```javascript 
// same as ES5.1
"𠮷".length == 2

// new RegExp behaviour, opt-in ‘u’
"𠮷".match(/./u)[0].length == 2

// new form
"\u{20BB7}" == "𠮷" == "\uD842\uDFB7"

// new String ops
"𠮷".codePointAt(0) == 0x20BB7

// for-of iterates code points
for(var c of "𠮷") {
  console.log(c);
}
``` 

## 模块
用于组件定义的模块的语言级支持。从流行的JavaScript模块加载器（AMD，CommonJS）中编码模式。由主机定义的默认加载程序定义的运行时行为。隐式异步模型 - 在请求的模块可用和处理之前，代码不会执行。

```javascript
// lib/math.js
export function sum(x, y) {
  return x + y;
}
export var pi = 3.141593;
```

```javascript
// app.js
import * as math from "lib/math";
console.log("2π = " + math.sum(math.pi, math.pi));
```

```javascript
// otherApp.js
import {sum, pi} from "lib/math";
console.log("2π = " + sum(pi, pi));
```

一些其他功能包括export default和export *：

```javascript
// lib/mathplusplus.js
export * from "lib/math";
export var e = 2.71828182846;
export default function(x) {
    return Math.exp(x);
}
```

```javascript
// app.js
import exp, {pi, e} from "lib/mathplusplus";
console.log("e^π = " + exp(pi));
```

> 模块格式化
> Babel可以将ES2015模块转换为几种不同的格式，包括Common.js，AMD，System和UMD。你甚至可以创建自己的。

## 模块加载
不属于ES2015
这保留在ECMAScript 2015规范中的实现定义中。最终标准将在WHATWG的Loader规范中，但这是目前正在进行的工作。以下内容来自之前的ES2015草案。

模块加载器支持：

> 动态加载
> 区域隔离
> 全局命名空间隔离
> 编译钩子
> 嵌套虚拟化

可以配置默认模块加载器，并且可以构造新的加载器以在隔离或受约束的上下文中调用和加载代码。

```javascript
// Dynamic loading – ‘System’ is default loader
System.import("lib/math").then(function(m) {
  alert("2π = " + m.sum(m.pi, m.pi));
});

// Create execution sandboxes – new Loaders
var loader = new Loader({
  global: fixup(window) // replace ‘console.log’
});
loader.eval("console.log(\"hello world!\");");

// Directly manipulate module cache
System.get("jquery");
System.set("jquery", Module({$: $})); // WARNING: not yet finalized
```

> 需要额外的polyfill
> 由于Babel默认使用common.js模块，因此它不包含模块加载器API的polyfill。

> 使用模块加载器
> 为了使用它，您需要告诉Babel使用system模块格式化程序。另请务必查看System.js

## Map + Set + WeakMap + WeakSet
常用算法的高效数据结构。WeakMaps提供无泄漏的对象密钥表。

```javascript
// Sets
var s = new Set();
s.add("hello").add("goodbye").add("hello");
s.size === 2;
s.has("hello") === true;

// Maps
var m = new Map();
m.set("hello", 42);
m.set(s, 34);
m.get(s) == 34;

// Weak Maps
var wm = new WeakMap();
wm.set(s, { extra: 42 });
wm.size === undefined

// Weak Sets
var ws = new WeakSet();
ws.add({ data: 42 });
// Because the added object has no other references, it will not be held in the set
```

## 代理
代理可以创建具有主机对象可用的全部行为的对象。可用于拦截，对象虚拟化，日志记录/分析等。

```javascript
// Proxying a normal object
var target = {};
var handler = {
  get: function (receiver, name) {
    return `Hello, ${name}!`;
  }
};

var p = new Proxy(target, handler);
p.world === "Hello, world!";
```

```javascript
// Proxying a function object
var target = function () { return "I am the target"; };
var handler = {
  apply: function (receiver, ...args) {
    return "I am the proxy";
  }
};

var p = new Proxy(target, handler);
p() === "I am the proxy";
```

所有运行时级元操作都有可用的方法：

```javascript
var handler =
{
  // target.prop
  get: ...,
  // target.prop = value
  set: ...,
  // 'prop' in target
  has: ...,
  // delete target.prop
  deleteProperty: ...,
  // target(...args)
  apply: ...,
  // new target(...args)
  construct: ...,
  // Object.getOwnPropertyDescriptor(target, 'prop')
  getOwnPropertyDescriptor: ...,
  // Object.defineProperty(target, 'prop', descriptor)
  defineProperty: ...,
  // Object.getPrototypeOf(target), Reflect.getPrototypeOf(target),
  // target.__proto__, object.isPrototypeOf(target), object instanceof target
  getPrototypeOf: ...,
  // Object.setPrototypeOf(target), Reflect.setPrototypeOf(target)
  setPrototypeOf: ...,
  // for (let i in target) {}
  enumerate: ...,
  // Object.keys(target)
  ownKeys: ...,
  // Object.preventExtensions(target)
  preventExtensions: ...,
  // Object.isExtensible(target)
  isExtensible :...
}
```

> 不支持的功能
> 由于ES5的限制，代理不能被转换或修改。请参阅各种JavaScript引擎中的支持。

## Symbols
Symbols启用对象状态的访问控制。符号允许属性被键入string（如在ES5中）或symbol。Symbols是一种新的原始类型。name调试中使用的可选参数 - 但不是Symbols的一部分。Symbols是独一无二的（如gensym），但不是私有的，因为它们通过反射功能暴露出来Object.getOwnPropertySymbols。

```javascript
(function() {

  // module scoped symbol
  var key = Symbol("key");

  function MyClass(privateData) {
    this[key] = privateData;
  }

  MyClass.prototype = {
    doStuff: function() {
      ... this[key] ...
    }
  };

  // Limited support from Babel, full support requires native implementation.
  typeof key === "symbol"
})();

var c = new MyClass("hello")
c["key"] === undefined
```

> 通过polyfill有限的支持
> 有限的支持需要Babel polyfill。由于语言限制，某些功能无法转换或修改。

## 子类可内置
在ES2015，内置插件一样Array，Date和DOM ElementS可被继承。

```javascript
// User code of Array subclass
class MyArray extends Array {
    constructor(...args) { super(...args); }
}

var arr = new MyArray();
arr[1] = 12;
arr.length == 2
```

> 部分支持
> 内置的子类可分性应该根据具体情况进行评估，因为类HTMLElement 可以被子类化，而许多例如Date，Array并且Error 不能归因于ES5引擎的限制。

## Math + Number + String + Object新增APIs
许多新的库添加，包括核心数学库，数组转换助手和用于复制的Object.assign。

```javascript
Number.EPSILON
Number.isInteger(Infinity) // false
Number.isNaN("NaN") // false

Math.acosh(3) // 1.762747174039086
Math.hypot(3, 4) // 5
Math.imul(Math.pow(2, 32) - 1, Math.pow(2, 32) - 2) // 2

"abcde".includes("cd") // true
"abc".repeat(3) // "abcabcabc"

Array.from(document.querySelectorAll("*")) // Returns a real Array
Array.of(1, 2, 3) // Similar to new Array(...), but without special one-arg behavior
[0, 0, 0].fill(7, 1) // [0,7,7]
[1,2,3].findIndex(x => x == 2) // 1
["a", "b", "c"].entries() // iterator [0, "a"], [1,"b"], [2,"c"]
["a", "b", "c"].keys() // iterator 0, 1, 2
["a", "b", "c"].values() // iterator "a", "b", "c"

Object.assign(Point, { origin: new Point(0,0) })
```

> 来自polyfill的有限支持
> Babel polyfill支持大多数这些API 。但是，由于各种原因省略了某些功能（例如， String.prototype.normalize需要许多额外的代码来支持）。

## 二进制和八进制
为binary（b）和octal（o）添加了两个新的数字文字形式。

```javascript
0b111110111 === 503 // true
0o767 === 503 // true
```

> 仅支持文字形式
> babel只能变换0o767而不能变换Number("0o767")。

## Promises
Promises是异步编程的库。Promise是可以在将来提供的值的第一类表示。Promises在许多现有的JavaScript库中使用。

```javascript
function timeout(duration = 0) {
    return new Promise((resolve, reject) => {
        setTimeout(resolve, duration);
    })
}

var p = timeout(1000).then(() => {
    return timeout(2000);
}).then(() => {
    throw new Error("hmm");
}).catch(err => {
    return Promise.all([timeout(100), timeout(200)]);
})
```

## Reflect API
Reflect API公开对象的运行时元操作。这实际上是Proxy API的反转，并允许进行与代理相同的元操作的调用。特别适用于实现代理。

```javascript
var O = {a: 1};
Object.defineProperty(O, 'b', {value: 2});
O[Symbol('c')] = 3;

Reflect.ownKeys(O); // ['a', 'b', Symbol(c)]

function C(a, b){
  this.c = a + b;
}
var instance = Reflect.construct(C, [20, 22]);
instance.c; // 42
```

## 尾递归调用
保证尾部位置的调用不会无限制地增加堆栈。在无界输入的情况下使递归算法安全。

```javascript
function factorial(n, acc = 1) {
    "use strict";
    if (n <= 1) return acc;
    return factorial(n - 1, n * acc);
}

// Stack overflow in most implementations today,
// but safe on arbitrary inputs in ES2015
factorial(100000)
```

完，原英文：https://github.com/lukehoban/es6features#readme。
