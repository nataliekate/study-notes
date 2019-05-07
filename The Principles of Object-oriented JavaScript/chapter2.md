# 函数（functions）

函数是对象。区别函数与其他对象的决定性特征是函数有一个内部属性（internal property）：`[[Call]]`。内部属性无法通过代码访问，而是定义了代码执行时的行为。

> `[[]]`**双重中括号**是ECMAScript为JavaScript定义的内部属性标注。

`[[Call]]`属性是函数**独有**的, 表明该对象可被执行。ECMAScript定义`typeof`操作符会对含有该内部属性的对象返回`"function"`

##  函数的创建
1. 字面形式（Literal Forms）

    函数有两种字面形式：函数声明（function declaration）和函数表达式（function expression）。

    * **函数声明**：以function关键字开头，后跟函数名。
      ```javascript
      function foo() {
        // content
      }
      ```
    * **函数表达式**：格式通常为匿名函数被一个变量或属性引用。
      ```javascript
      var foo = function() {
        // content
      };
      ```

    > 函数声明会被提升（hoisted）至上下文（该函数被声明时所在函数的范围，或全局范围）的顶部，因为引擎提前知道了函数的名字，这意味着可以先使用函数后声明。而函数表达式仅能通过变量引用，因此无法提升。

    [变量提升(MDN)](https://developer.mozilla.org/zh-CN/docs/Glossary/Hoisting)

2. `Function`构造函数

    可以用来创建新函数，但不建议。因为会使代码难以理解和调试。

> 函数可以作为对象使用，也可以作为值，将他们赋给变量、在对象中添加、当成参数传递给其他函数，或从其他函数中返回。基本只要是可以使用引用值的地方，就可以使用函数。

## 参数（Parameters）

JS函数的参数保存在一个叫做`arguments`的**类数组（array-like）对象** 中，它可以自由增长，其中的值可以通过数字索引（index）来使用。因此向函数传递任意个数的参数都不会抛出错误。arguments对象是所有（非箭头）函数中都可用的**局部变量**。

> Note: “Array-like” means that arguments has a `length` property and properties indexed from zero, but it doesn't have Array's built-in methods like `forEach()` and `map()`. [来源：MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments)

```javascript
console.log(typeof arguments); // 'object'
```

要把`arguments`转换成真正的数组对象，可以使用以下方法：
```javascript
var args = Array.prototype.slice.call(arguments);
// Using an array literal is shorter than above but allocates an empty array
var args = [].slice.call(arguments);

// ES2015
var args = Array.from(arguments);
var args = [...arguments];
```

> 如果是ES6兼容代码，推荐使用[...剩余参数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)


通过`arguments.length`属性可以获取**实际**传入参数的个数。

函数的**期望**参数个数保存在**函数**的`length`属性内。

实际传入的参数个数不影响函数的期望参数个数。

```javascript
function foo(){
  return arguments.length;
}

console.log(foo(1,2,3,4));    // 4
console.log(foo.length);      // 0
```

### 什么时候使用`arguments`？

如果调用的参数多于正式声明接受的参数，则可以使用`arguments`对象。这种技术对于可以传递可变数量的参数的函数（如sum，min，max）很有用。使用 `arguments.length` 来确定传递给函数参数的个数，然后使用`arguments`对象来处理每个参数。
[例子(MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/arguments#Examples)

## 重载（function overloading）

大多数面向对象语言支持函数重载，它让一个函数具有多个签名（signature，包含函数名，参数个数和类型），这些语言根据实际传入的参数决定调用函数的版本。

而JavaScript函数没有签名，不存在重载。但我们可以使用`arguments`对象模仿函数重载。
```javascript
function logValue(value) {
  if(arguments.length === 0) {
    value = "default value";
  }

  console.log(value);
}

logValue("2333"); // 输出2333
```
> 实际使用中，检查命名参数是否为`undefined`比依赖`arguments.length`更常见。

## 对象方法（object methods）

当函数是对象的属性值，则该属性被称为方法。

## `this`对象

所有函数的作用域内都有一个this对象代表其执行时的环境（context）对象。

1. 全局环境

    无论是否在严格模式（strict mode）下，在全局执行环境中（在任何函数体外部）this 都指向全局对象（即浏览器的`window`）。
    ```javascript
    // 在浏览器中, window 对象同时也是全局对象：
    console.log(this === window); // true
    ```

2. 函数环境
    
    在函数内部，this的值取决于函数被调用的方式。如果作为对象方法被调用，默认`this`为该对象。

    ```javascript
    // 非严格模式下
    function f1(){
      return this;
    }

    f1() === window;   // 在浏览器中，全局对象是window
    f1() === global;   // 在node中，全局对象是global



    // 严格模式下
    function f2(){
      "use strict"; // 这里是严格模式
      return this;
    }

    f2() === undefined; // true
    ```

3. 箭头函数

    在箭头函数中，this与封闭词法环境（enclosing lexical context）的this保持一致。箭头函数不会创建自己的this，它只会从自己的作用域链的上一层继承this

    ```javascript
    // 全局环境
    var globalObject = this;
    var foo = (() => this);
    console.log(foo() === globalObject); // true

    // 函数环境
    function Person(){
      this.age = 0;

      setInterval(() => {
        this.age++; // |this| 正确地指向 p 实例
      }, 1000);
    }

    var p = new Person();
    ```

## 改变`this`

有三种函数方法（继承自`Function.prototype`）可以改变`this`的值。

1. `call()`

    该方法使用一个指定的 `this` 值和单独给出的一个或多个参数来调用一个函数
    
    语法：

    `fun.call(thisArg, arg1, arg2, ...);`

    注意调用函数时函数名后没有小括号，因为它是被作为对象访问，而不是被执行的代码。

2. `apply()`

    语法和作用与 `call()` 方法类似，只有一个区别，就是 `call()` 方法接受的是一个**参数列表**，而 `apply()` 方法接受的是一个包含多个参数的**数组或类数组对象**。

    语法：
    
    `func.apply(thisArg, [argsArray])`

> 如果你已经有一个数组，用`apply()`， 如果你只有一个个单独的变量，用`call()`

3. `bind()`

    ECMAScript 5 新增的方法。

    该方法创建一个新的函数，在调用时设置`this`关键字为提供的值。并在调用新函数时，将给定参数列表作为原函数的参数序列的**前若干项**。

    [例子](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind#%E7%A4%BA%E4%BE%8B)
