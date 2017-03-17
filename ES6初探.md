# ES6初探（待更新）  
## let,const与var的比较  
转载自阮一峰老师的[ES6入门](http://es6.ruanyifeng.com)，稍有修改  
### 1. 基本概念[1]  
var声明了一个变量，并且可以同时初始化该变量。  
let语句声明一个块级作用域的本地变量，并且可选的赋予初始值。  
const 声明创建一个只读的常量，作用域与let相同。这不意味着常量指向的值不可变，而是变量标识符的值只能赋值一次。  
### 2.基本用法  
var为大家所熟悉，在这里不再赘述。  
#### let
let的用法类似于var，但是所声明的变量，只在let命令所在的代码块内有效。

    {  
      let a = 10;  
      var b = 1;  
    }  
    a // ReferenceError: a is not defined.  
    b // 1
    
上面代码在代码块之中，分别用let和var声明了两个变量。然后在代码块之外调用这两
个变量，结果let声明的变量报错，var声明的变量返回了正确的值。这表明，let声明
的变量只在它所在的代码块有效。 

for循环的计数器，就很合适使用let命令。

    for (let i = 0; i < 10; i++) {}
    
    console.log(i);
    //ReferenceError: i is not defined
    
上面代码中，计数器i只在for循环体内有效，在循环体外引用就会报错。

另外，for循环还有一个特别之处，就是循环语句部分是一个父作用域，而循环体内部
是一个单独的子作用域。

    for (let i = 0; i < 3; i++) {
      let i = 'abc';
      console.log(i);
    }
    // abc
    // abc
    // abc
    
上面代码输出了3次abc，这表明函数内部的变量i和外部的变量i是分离的。  
#### const  
const声明一个只读的常量。一旦声明，常量的值就不能改变。

    const PI = 3.1415;
    PI // 3.1415
    
    PI = 3;
    // TypeError: Assignment to constant variable.
    
上面代码表明改变常量的值会报错。

const声明的变量不得改变值，这意味着，const一旦声明变量，就必须立即初始化，不能留到以后赋值。

    const foo;
    // SyntaxError: Missing initializer in const declaration
    
上面代码表示，对于const来说，只声明不赋值，就会报错。

const的作用域与let命令相同：只在声明所在的块级作用域内有效。
  
### 3.变量声明提升与暂时性死区  
#### 变量声明提升[1]  
>###### 此段仅适用var声明  
>由于变量声明（以及其他声明）总是在任意代码执行之前处理的，所以在代码中的任
意位置声明变量总是等效于在代码开头声明。这意味着变量可以在声明之前使用，这
个行为叫做“hoisting”。“hoisting”就像是把所有的变量声明移动到函数或者全
局代码的开头位置。

    bla = 2
    var bla;
    // ...
    // 可以理解为：   
    var bla;  
    bla = 2;

>由于这个原因，我们建议总是在作用域的最开始（函数或者全局代码的开头）声明变
量。这样可以使变量的作用域变得清晰。

var命令会发生”变量提升“现象，即变量可以在声明之前使用，值为undefined。这种
现象多多少少是有些奇怪的，按照一般的逻辑，变量应该在声明语句之后才可以使用。

为了纠正这种现象，let命令改变了语法行为，它所声明的变量一定要在声明后使用，
否则报错。

    // var 的情况
    console.log(foo); // 输出undefined
    var foo = 2;
    
    // let 的情况
    console.log(bar); // 报错ReferenceError
    let bar = 2;
    
上面代码中，变量foo用var命令声明，会发生变量提升，即脚本开始运行时，变量foo
已经存在了，但是没有值，所以会输出undefined。变量bar用let命令声明，不会发生
变量提升。这表示在声明它之前，变量bar是不存在的，这时如果用到它，就会抛出一
个错误。

const的作用域与let命令相同：只在声明所在的块级作用域内有效。

    if (true) {
      const MAX = 5;
    }
    
    MAX // Uncaught ReferenceError: MAX is not defined
    
#### 暂时性死区  
只要块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个区域，
不再受外部的影响。

    var tmp = 123;
    
    if (true) {
      tmp = 'abc'; // ReferenceError
      let tmp;
    }
    
上面代码中，存在全局变量tmp，但是块级作用域内let又声明了一个局部变量tmp，导致后者绑定这个块级作用域，所以在let声明变量前，对tmp赋值会报错。

ES6明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

总之，在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

    if (true) {
      // TDZ开始
      tmp = 'abc'; // ReferenceError
      console.log(tmp); // ReferenceError
    
      let tmp; // TDZ结束
      console.log(tmp); // undefined
    
      tmp = 123;
      console.log(tmp); // 123
    }

上面代码中，在let命令声明变量tmp之前，都属于变量tmp的“死区”。

“暂时性死区”也意味着typeof不再是一个百分之百安全的操作。

    typeof x; // ReferenceError
    let x;
    
上面代码中，变量x使用let命令声明，所以在声明之前，都属于x的“死区”，只要用到该变量就会报错。因此，typeof运行时就会抛出一个ReferenceError。

作为比较，如果一个变量根本没有被声明，使用typeof反而不会报错。

    typeof undeclared_variable // "undefined"

上面代码中，undeclared_variable是一个不存在的变量名，结果返回“undefined”。所以，在没有let之前，typeof运算符是百分之百安全的，永远不会报错。现在这一点不成立了。这样的设计是为了让大家养成良好的编程习惯，变量一定要在声明之后使用，否则就报错。


另外，下面的代码也会报错，与var的行为不同。

    // 不报错
    var x = x;
    
    // 报错
    let x = x;
    // ReferenceError: x is not defined

上面代码报错，也是因为暂时性死区。使用let声明变量时，只要变量在还没有声明完成前使用，就会报错。上面这行就属于这个情况，在变量x的声明语句还没有执行完成前，就去取x的值，导致报错”x 未定义“。

ES6 规定暂时性死区和let、const语句不出现变量提升，主要是为了减少运行时错误，防止在变量声明前就使用这个变量，从而导致意料之外的行为。这样的错误在 ES5 是很常见的，现在有了这种规定，避免此类错误就很容易了。


总之，暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。

#### 4.重复声明  
var是允许在相同作用域内重复声明同一个变量的，而let与const不允许这一现象。

    // 报错
    function () {
      let a = 10;
      var a = 1;
    }
    
    // 报错
    function () {
      let a = 10;
      let a = 1;
    }

因此，不能在函数内部重新声明参数。

    function func(arg) {
      let arg; // 报错
    }
    
    function func(arg) {
      {
        let arg; // 不报错
      }
    }
    
const声明的常量，也与let一样不可重复声明。

    var message = "Hello!";
    let age = 25;
    
    // 以下两行都会报错
    const message = "Goodbye!";
    const age = 30;


[1]:https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements "MDN 语句和声明"