<!--
 * @Author: yyt
 * @Date: 2020-05-26 11:00:16
 * @LastEditTime: 2020-05-26 17:33:04
 * @LastEditors: yyt
 * @FilePath: /taoNote/doc/knowledge/hard/W3C/JavaScript/runtime/dataType.md
-->

## JavaScript 类型

- 7 种语言类型
  - Undefined
    - undefined 是一个`变量`，而并非是一个关键字，这是 JavaScript 语言公认的设计失误之一，所以，我们为了避免无意中被篡改，我建议使用 `void 0` 来获取 undefined 值。
    - 在 ES5 之前的时候，全局 undefined 是可以被赋值的。在现代浏览器当中已经把 全局 undefined 设置为一个 non-configurable, non-writable 属性的值了。但是局部 undefined 的依然可以赋值
    - `(()=>{let undefined=1;console.log(undefined)})()//1`
  - Null
    - null 是定义了，但是值是空
    - undefined 是未定义
  - Boolean
  - String
    - Note：现行的字符集国际标准，字符是以 Unicode 的方式表示的，每一个 Unicode 的码点表示一个字符，理论上，Unicode 的范围是无限的。UTF 是 Unicode 的编码方式，规定了码点在计算机中的表示方法，常见的有 UTF16 和 UTF8。 Unicode 的码点通常用 U+??? 来表示，其中 ??? 是十六进制的码点值。 0-65536（U+0000 - U+FFFF）的码点被称为基本字符区域（BMP）。
  - Number
    - 正确的比较方法是使用 JavaScript 提供的最小精度值： `console.log( Math.abs(0.1 + 0.2 - 0.3) <= Number.EPSILON)`;
  - Symbol
    - 创建： var mySymbol = Symbol("my symbol");类型
  - Object
    - JavaScript 中的几个基本类型，都在对象类型中有一个“亲戚”。它们是：
      - Number；
      - String；
      - Boolean；
      - Symbol
      - 所以，我们必须认识到 3 与 new Number(3) 是完全不同的值，它们一个是 Number 类型， 一个是对象类型。
- 判断类型
  - typeof
    - undefined
    - boolean
    - string
    - number
      - 30；二进制 0b111；八进制 0o13；十六进制 0xFF。
      - 科学计数法 1e3；-1e-2。
    - object-----对象或者 null...正则 RegExp 也是
    - function
    - Symbol
  - instanceof 可以用于 typeof 之后的区分 obj 是对象，数组，null，还是正则
    - `some instanceof Object/Array/RegExg`
    - `instanceof Array`区分数组和对象
- 类型转换

  - string=>number
    - parseInt 第二个参数为进制，parseFloat 只能 10 进制
  - 这里我们当然也不打算讲解 == 的规则，它属于设计失误，并非语言中有价值的部分，很多实践中推荐禁止使用“ ==”，而要求程序员进行显式地类型转换后，用 === 比较。
  - 装箱转换:把基本类型转换为对应的对象

  ```js
  var symbolObject = function () {
    return this;
  }.call(Symbol("a"));
  console.log(typeof symbolObject); //object
  console.log(symbolObject instanceof Symbol); //true
  console.log(symbolObject.constructor == Symbol); //true

  //装箱机制会频繁产生临时对象，在一些对性能要求较高的场景下，我们应该尽量避免对基本类型做装箱转换。
  //使用内置的 Object 函数，我们可以在 JavaScript 代码中显式调用装箱能力。
  var symbolObject = Object(Symbol("a"));
  console.log(typeof symbolObject); //object
  console.log(symbolObject instanceof Symbol); //true
  console.log(symbolObject.constructor == Symbol); //true
  ```

  - 拆箱转换:对象类型到基本类型的转换

  ```js
  //到number
  var o = {
    valueOf: () => {
      console.log("valueOf");
      return {};
    },
    toString: () => {
      console.log("toString");
      return {};
    },
  };
  o * 2;
  // valueOf 先
  // toString
  // TypeError
  ```


        //到string
        var o = {
            valueOf : () => {console.log("valueOf"); return {}},
            toString : () => {console.log("toString"); return {}}
        }

        String(o)
        // toString  先
        // valueOf
        // TypeError

    ```

- 但 JS 之父本人也在多个场合表示过，typeof 的设计是有缺陷的，只是现在已经错过了修正它的时机。
