# 理解对象

JavaScript中的对象是动态的（dynamic），可在代码执行的任意时刻发生改变。

## 属性 (Property)

### 1. 定义/删除属性：

* 使用点(`.`)或中括号(`[]`)来访问/定义属性

  ```javascript
  // dot notation
  var person1 = {
    name: "natalie"
  };

  var person2 = new Object();
  person2.name = "natalie";
  ```

  例子中 `name` 为 `person1`，`person2` 的属性。

* 使用 `Object.defineProperty()` 或 `Object.defineProperties()`来创建/修改属性

  ```javascript
  var person = {};

  Object.defineProperties(person, {
    // 定义一个数据属性
    _name: {
      value: "natalie",
      enumerable: true,
      configurable: true,
      writable: true
    },

    // 定义一个访问器属性
    name: {
      get: function() {
        console.log("Reading name");
        return this._name;
      },
      set: function(value) {
        console.log("Setting name to %s", value);
        this._name = value;
      },
      enumerable: true,
      configurable: true
    }
  });

  person.name;
  // Reading name
  // "natalie" 

  person.name = "foo";
  // Setting name to foo
  // "foo"
  ```

  当一个属性**第一次**添加到对象中，该对象的内部方法 `[[Put]]` 被调用。该方法会在对象中创建一个新节点来保存这个属性。使用这个方法创建的属性是对象的**自有属性 (own property)**，与原型属性 (prototype property) 区别。

  > 自有属性：只有该对象的实例才拥有的属性

  当已有属性被赋予新值时，内部方法 `[[Set]]` 被调用。该方法将属性的当前值替换为新值。

  <!-- ![Adding and changing properties of an object](https://github.com/nataliekate/study-notes/raw/master/assets/add_prop.png) -->

* 使用 `delete` 操作符来彻底移除属性

  <!-- ![Adding and changing properties of an object](https://github.com/nataliekate/study-notes/raw/master/assets/delete_prop.png) -->

  `delete` 针对单个对象属性调用内部方法 `[[Delete]]`，操作成功时返回 `true`


### 2. 探测属性 (Detecting Properties)

检查对象是否包含某属性的方法有：

* 属性访问器(.)
    
    ```javascript
    // unreliable 不可靠的方法
    // 当 if 判断中的值为 truthy 时，判断为真；若值为 falsy，则判断为假。
    if(person.age) {
      // do something
    }
    ```
  **Truthy and Falsy:**
  
    以下值总为Falsy：
    * `false`
    * `0` (zero)
    * `''` or `""` (empty string)
    * `null`
    * `undefined`
    * `NaN`

    以下值总为Truthy：
    * `'0'` (a string containing a single zero)
    * `'false'` (a string containing the text “false”)
    * `[]` (an empty array)
    * `{}` (an empty object)
    * `function(){}` (an “empty” function)

    使用这个方法的弊端是，当`age`的值为falsy时，就会出现判断错误的情况。

* `in` 操作符

  The `in` operator returns `true` if the specified property is in the specified **object** or its **prototype chain**.
  
  [Examples(MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/in#Description)

  ```javascript
  var car = {make: 'Honda', model: 'Accord', year: 1998};

  console.log('make' in car);
  // expected output: true

  delete car.make;
  if ('make' in car === false) {
    car.make = 'Suzuki';
  }

  console.log(car.make);
  // expected output: "Suzuki"
  ```

* `hasOwnProperty()`

  `in` 会检查自有属性和原型属性，如果仅希望检查自有属性，则应该使用 `hasOwnProperty` 方法。

  ```javascript
  const object1 = new Object();
  object1.property1 = 42;

  console.log(object1.hasOwnProperty('property1'));
  // expected output: true, own property

  console.log(object1.hasOwnProperty('toString'));
  // expected output: false, prototype property

  console.log(object1.hasOwnProperty('hasOwnProperty'));
  // expected output: false, prototype property
  ```

  > `hasOwnProperty` returns `true` even if the value of the property is `null` or `undefined`.

### 3. 属性类型 (Types of Properties)

  属性有两种类型：**数据属性** (data properties) 和**访问器属性** (accessor properties)。

  **数据属性**包含一个值，`[[Put]]`方法的默认行为时创建数据属性。

  **访问器属性**不保存值，用 `getter`（读取时调用）函数和 `setter`（写入时调用）函数来进行制定操作。

  ```javascript
  // define getter/setter using an object initializer
  var o1 = {
    a: 7,
    get b() {               // 没有function关键字
      return this.a + 1;
    },
    set c(x) {              // 没有function关键字
      this.a = x / 2;
    }
  };

  console.log(o1.a); // 7
  console.log(o1.b); // 8
  o1.c = 50;
  console.log(o1.a); // 25

  // using Object.defineProperties method
  var o2 = { a: 0 };

  Object.defineProperties(o2, {
      'b': { get: function() { return this.a + 1; } },
      'c': { set: function(x) { this.a = x / 2; } }
  });

  o2.c = 10; // Runs the setter, which assigns 10 / 2 (5) to the 'a' property
  console.log(o2.b); // Runs the getter, which yields a + 1 or 6
  
  ```

### 4. 属性特征 (Property Attributes)

* **通用特征 (common attributes)**：数据属性和访问器熟悉都具有 `[[Enumerable]]` (可枚举，可遍历) 和 `[[Configurable]]`（可修改/删除）。默认均为`true`

* **数据属性特征**：`[[Value]]` 属性的值，`[[Writable]]`（布尔值）表示属性是否可写入。默认均为`true`

* **访问器属性特征**：`[[Get]]`对应 `getter` 函数以及 `[[Set]]` 对应 `setter` 函数

> 当使用 `Object.defineProperty()` 定义新属性时，必须为特征指定一个值，否则*布尔型*的特征会被默认设置为 `false`

* **获取属性特征**：使用 `Object.getOwnPropertyDescriptor(obj, propName)` 方法

* **属性的枚举 (Enumeration)**： 当对象的属性是可枚举的（`[[Enumerable]]` 为 `true`），就可以遍历。

  从**ECMAScript 5**开始，有三种原生的方法用于列出或枚举对象的属性：

  * `for...in` 循环

    该方法依次访问一个**对象及其原型链中**所有可枚举的属性。
  
  * `Object.keys(o)`
  
    该方法返回一个对象 `o` 所有**自有属性**的名称的数组。
  
  * `Object.getOwnPropertyNames(o)`
    
    该方法返回一个数组，它包含了对象 `o` 所有拥有的属性（**无论是否可枚举**）的名称。

> 使用对象的 `propertyIsEnumerable(propertyName)` 方法来检查一个属性是否可枚举

### 4. 锁定对象

对象具有 `[[Extensible]]`（布尔值）特征，表明对象是否可以修改。所有对象默认是可扩展的。

* **禁止扩展**(Preventing Extensions)
    
    使用 `Object.preventExtensions()` 创建一个不可扩展的对象。
    
    使用 `Object.isExtensible()` 来检查是否可扩展。

    > 注意，一般来说，不可扩展对象的属性可能仍然可被删除。尝试将新属性添加到不可扩展对象将静默失败或抛出 `TypeError`（最常见但不排除其他情况，如在strict mode中）。

    > `Object.preventExtensions()` 仅阻止添加自身的属性。但属性仍然可以添加到对象原型。

* **封印**(Seal)

  被封印的对象不可扩展，且其所有属性都不可配置。当前属性的值只要可写就可以改变。但是属性不能被删除，或改变类型。

  方法：
  ```javascript
  // object's [[Extensible]] -> false
  // properties' [[Configurable]] -> false
  Object.seal(o);
  Object.isSealed(o);
  ```

* **冻结**(Freeze)

  对象被冻结后，不能添加或删除属性，不能改变属性类型，也不能写入任何数据属性(浅冻结)。对象为只读，无法解冻。冻结一个对象后该对象的**原型也不能被修改**。`freeze()` 返回和传入的参数相同的对象。

  ```javascript
  Object.freeze(o);   // can be used on Array
  Object.isFrozen(o);
  ```

  例外：
  
  ```javascript
  var obj1 = {
    internal: {}
  };

  Object.freeze(obj1);
  obj1.internal.a = 'aValue';

  obj1.internal.a // 'aValue'
  ```