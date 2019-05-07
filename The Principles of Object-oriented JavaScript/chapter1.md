面向对象的语言有如下几种特性：
* 封装（Encapsulation）：数据可以和操作数据的功能组织在一起，即对象的定义
* 聚合（Aggregation）：一个对象能够引用另一个对象
* 继承（Inheritance）：一个新创建的对象和另一个对象拥有同样的特性，而无需显示复制其功能
* 多态（Polymorphism）：一个接口可以被多个对象实现

## JavaScript数据类型
JavaScripts虽没有类的概念，但依然存在两种*类型（types）*：原始类型（primitive）和引用类型（reference）
* *原始类型（primitive types）*  保存为简单数据值（simple data types）
* *引用类型（reference types）*  则保存为对象（objects），本质是指向内存位置的引用（references to locations in memory）

> ECMAScript 2015 中引入了 [JavaScript 类](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)，实质上是 JavaScript 现有的基于原型的继承的语法糖。类语法不会为JavaScript引入新的面向对象的继承模型。

## 原始类型（Primitive Types）
JavaScript共有6种原始数据类型：
boolean       `true`or `false` 
number        Any integer or floating-point numeric value 
string            A character or sequence of characters delimited by either single or double quotes (JavaScript has no separate character type)
null               Has only one value, `null`
undefined   Has only one value, `undefined` (undefined is the value assigned to a variable that is not initialized)
symbol（ECMAScript 2015新增） A token that serve as unique ID. You create symbols via the factory function `Symbol()`

所有原始类型的值都有字面表达形式（literal representations）。  
*字面形式（Literals）* 是不被保存在变量中的值，如hardcoded的值。
```javascript
// strings
var name = "Nicholas";

// count
var count = 25;

// boolean
var found = true;

// null
var object = null;

// undefined
var flag = undefined;
var ref; // automatically assigned undefined
```

## 引用类型（Reference Types）
引用类型指的是JS中的对象（objects），引用值是引用类型的实例，也是对象的同义词。
Reference values are *instances* of reference types and are synonymous with objects

对象是 *属性* 的无序列表。属性包含键和值。
An object is an unordered list of /properties/ consisting of a name (always a string) and a value.  

如果一个属性的值是函数，他就被称为 _方法(method)_ 。

### 创建/实例化(instantiate)对象的方法 
[方法及prototype参考](https://codeburst.io/various-ways-to-create-javascript-object-9563c6887a47)
1. 使用`new`操作符和构造函数(constructor)，根据命名规范，构造函数首字母大写来跟非构造函数进行区分。
    ```javascript
    function Vehicle(name, maker) {
      this.name = name;
      this.maker = maker;
    }
    let car1 = new Vehicle(’Fiesta’, 'Ford’);
    let car2 = new Vehicle(’Santa Fe’, 'Hyundai’)
    console.log(car1.name);    //Output: Fiesta
    console.log(car2.name);    //Output: Santa Fe
    ```

2. 使用对象字面形式（/object literal/），使用dot notation和中括号访问/创建属性。
    ```javascript
    let bike = {
      name: 'SuperSport', 
      maker:'Ducati', 
      start: function() {
          console.log('Starting the engine...');
      }
    };
    bike.start();   //Output: Starting the engine...
    /* Adding method stop() later to the object */
    bike.stop = function() {
        console.log('Applying Brake...');  
    }
    bike.stop();    //Output: Applying Brake...
    ```

4.  `Object.create()`方法

    ```javascript
    let car = Object.create(Object.prototype,
      {
        name:{
          value: 'Fiesta',
          configurable: true,
          writable: true,
          enumerable: false
        },
        maker:{
          value: 'Ford',
          configurable: true,
          writable: true,
          enumerable: true
        }
      });
    console.log(car.name)    //Output: Fiesta
    ```

4. 使用ES6的class
    ```javascript
    class Vehicle {
      constructor(name, maker, engine) {
        this.name = name;
        this.maker =  maker;
        this.engine = engine;
      }
      start() {
        console.log("Starting...");
      }
    }
    let bike = new Vehicle('Hayabusa', 'Suzuki', '1340cc');
    bike.start();    //Output: Starting...
    /* Adding method brake() later to the created object */
    bike.brake = function() {
      console.log("Applying Brake...");
    }
    bike.brake();    //Output: Applying Brake...
    ```

## 鉴别数据类型
1. 鉴别*原始类型*最佳方法是使用`typeof`操作符，如字符串，数字，布尔和undefined
2. 鉴别`null`类型，由于`typeof null === "object"`，判断一个值是否为空的最佳方法是*直接和null比较*
    ```javascript
    console.log(value === null); // true or false
    /* comparing without coercion
    * 使用三等号而不是双等号
    * 双等号会将变量强制转换为另一种类型
    */
    console.log("5" == 5); // true
    console.log("5" === 5); // false
    ```

3. 鉴别引用类型：
	* Function可使用`typeof`进行鉴别：
      ```javascript
      function reflect(value){
        return value;
      }

      console.log(typeof reflect); // "function"
      ```

	* 非函数的引用类型，使用`typeof`会返回 “object”，可以使用`instanceof`操作符：
      ```javascript
      var items = [];
      var object = {};

      console.log(items instanceof Array) // true
      console.log(object instanceof Object) // true
      ```

## 原始封装类型（Primitive Wrapper Types）

为了让原始类型看上去更像引用类型，JavaScript提供了三种原始封装类型：`String` 、 `Number` 和 `Boolean` 。JavaScript会在背后创建这些对象，使得原始值能像对象一样被使用。当语句结束时，这些临时对象就会立即被销毁。

当读取**字符串**、**数字**或**布尔值**时，原始封装类型将被自动创建。

例子：
对字符串使用`charAt()`方法
```javascript
var name = "Nicholas";
var firstChar = name.charAt(0);

console.log(firstChar); // "N"
```

实际上JS引擎做的事
```javascript
var name = "Nicholas";
var temp = new String(name); // 创建String对象
var firstChar = temp.charAt(0);

temp = null; // 销毁临时对象, 

console.log(firstChar); // "N"
```

**自动打包：** The `String` object exists only for one statement before it’s destroyed (a process called *autoboxing*). 

临时对象仅在值被读取时创建（a temporary object is
created only when a value is read）

