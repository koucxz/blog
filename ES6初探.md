# ES6初探（待更新）  
转载自阮一峰老师的[ES6入门](http://es6.ruanyifeng.com)，稍有修改  
## let,const与var的比较  

### 1. 基本概念[1]  
var声明了一个变量，并且可以同时初始化该变量。  
let语句声明一个块级作用域的本地变量，并且可选的赋予初始值。  
const 声明创建一个只读的常量，作用域与let相同。这不意味着常量指向的值不可变，而是变量标识符的值只能赋值一次。  
#### 块级作用域
let实际上为 JavaScript 新增了块级作用域。

    function f1() {
      let n = 5;
      if (true) {
        let n = 10;
      }
      console.log(n); // 5
    }

上面的函数有两个代码块，都声明了变量n，运行后输出5。这表示外层代码块不受内层代码块的影响。如果使用var定义变量n，最后输出的值就是10。  
块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了。

    // IIFE 写法
    (function () {
      var tmp = ...;
      ...
    }());
    
    // 块级作用域写法
    {
      let tmp = ...;
      ...
    }

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

## 模板字符串  
### 1.基本用法  
传统的JavaScript语言，输出模板通常是这样写的。  

    $('#result').append(
      'There are <b>' + basket.count + '</b> ' +
      'items in your basket, ' +
      '<em>' + basket.onSale +
      '</em> are on sale!'
    );
上面这种写法相当繁琐不方便，ES6引入了模板字符串解决这个问题。

    $('#result').append(`
             There are <b>${basket.count}</b> items
              in your basket, <em>${basket.onSale}</em>
             are on sale!
           `);
模板字符串（template string）是增强版的字符串，用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。

// 普通字符串  

    `In JavaScript '\n' is a line-feed.`

// 多行字符串

    `In JavaScript this is
     not legal.`
    
    console.log(`string text line 1
    string text line 2`);

// 字符串中嵌入变量

    var name = "Bob", time = "today";
    `Hello ${name}, how are you ${time}?`
    
上面代码中的模板字符串，都是用反引号表示。如果在模板字符串中需要使用反引号，则前面要用反斜杠转义。

    var greeting = `\`Yo\` World!`;
    
如果使用模板字符串表示多行字符串，所有的空格和缩进都会被保留在输出之中。
    
    $('#list').html(`
    <ul>
      <li>first</li>
      <li>second</li>
    </ul>
    `);
    
上面代码中，所有模板字符串的空格和换行，都是被保留的，比如<ul>标签前面会有一个换行。如果你不想要这个换行，可以使用trim方法消除它。

    $('#list').html(`
    <ul>
      <li>first</li>
      <li>second</li>
    </ul>
    `.trim());
    
模板字符串中嵌入变量，需要将变量名写在${}之中。

    function authorize(user, action) {
      if (!user.hasPrivilege(action)) {
        throw new Error(
          // 传统写法为
          // 'User '
          // + user.name
          // + ' is not authorized to do '
          // + action
          // + '.'
          `User ${user.name} is not authorized to do ${action}.`);
      }
    }
    
大括号内部可以放入任意的JavaScript表达式，可以进行运算，以及引用对象属性。

    var x = 1;
    var y = 2;
    
    `${x} + ${y} = ${x + y}`
    // "1 + 2 = 3"
    
    `${x} + ${y * 2} = ${x + y * 2}`
    // "1 + 4 = 5"
    
    var obj = {x: 1, y: 2};
    `${obj.x + obj.y}`
    // 3
    
模板字符串之中还能调用函数。

    function fn() {
      return "Hello World";
    }
    
    `foo ${fn()} bar`
    // foo Hello World bar
    
如果大括号中的值不是字符串，将按照一般的规则转为字符串。比如，大括号中是一个对象，将默认调用对象的toString方法。

如果模板字符串中的变量没有声明，将报错。

    // 变量place没有声明
    var msg = `Hello, ${place}`;
    // 报错
    
由于模板字符串的大括号内部，就是执行JavaScript代码，因此如果大括号内部是一个字符串，将会原样输出。

    `Hello ${'World'}`
    // "Hello World"
    
模板字符串甚至还能嵌套。

    const tmpl = addrs => `
      <table>
      ${addrs.map(addr => `
        <tr><td>${addr.first}</td></tr>
        <tr><td>${addr.last}</td></tr>
      `).join('')}
      </table>
    `;
    
上面代码中，模板字符串的变量之中，又嵌入了另一个模板字符串，使用方法如下。

    const data = [
        { first: '<Jane>', last: 'Bond' },
        { first: 'Lars', last: '<Croft>' },
    ];
    
    console.log(tmpl(data));
    // <table>
    //
    //   <tr><td><Jane></td></tr>
    //   <tr><td>Bond</td></tr>
    //
    //   <tr><td>Lars</td></tr>
    //   <tr><td><Croft></td></tr>
    //
    // </table>
    
如果需要引用模板字符串本身，在需要时执行，可以像下面这样写。

    // 写法一
    let str = 'return ' + '`Hello ${name}!`';
    let func = new Function('name', str);
    func('Jack') // "Hello Jack!"
    
    // 写法二
    let str = '(name) => `Hello ${name}!`';
    let func = eval.call(null, str);
    func('Jack') // "Hello Jack!"
    
### 2.实例：模板编译  
下面，我们来看一个通过模板字符串，生成正式模板的实例。

    var template = `
    <ul>
      <% for(var i=0; i < data.supplies.length; i++) { %>
        <li><%= data.supplies[i] %></li>
      <% } %>
    </ul>
    `;
    
上面代码在模板字符串之中，放置了一个常规模板。该模板使用<%...%>放置JavaScript代码，使用<%= ... %>输出JavaScript表达式。

怎么编译这个模板字符串呢？

一种思路是将其转换为JavaScript表达式字符串。

    echo('<ul>');
    for(var i=0; i < data.supplies.length; i++) {
      echo('<li>');
      echo(data.supplies[i]);
      echo('</li>');
    };
    echo('</ul>');
    
这个转换使用正则表达式就行了。

    var evalExpr = /<%=(.+?)%>/g;
    var expr = /<%([\s\S]+?)%>/g;
    
    template = template
      .replace(evalExpr, '`); \n  echo( $1 ); \n  echo(`')
      .replace(expr, '`); \n $1 \n  echo(`');
    
    template = 'echo(`' + template + '`);';
然后，将template封装在一个函数里面返回，就可以了。
    
    var script =
    `(function parse(data){
      var output = "";
    
      function echo(html){
        output += html;
      }
    
      ${ template }
    
      return output;
    })`;
    
    return script;
将上面的内容拼装成一个模板编译函数compile。
    
    function compile(template){
      var evalExpr = /<%=(.+?)%>/g;
      var expr = /<%([\s\S]+?)%>/g;
    
      template = template
        .replace(evalExpr, '`); \n  echo( $1 ); \n  echo(`')
        .replace(expr, '`); \n $1 \n  echo(`');
    
      template = 'echo(`' + template + '`);';
    
      var script =
      `(function parse(data){
        var output = "";
    
        function echo(html){
          output += html;
        }
    
        ${ template }
    
        return output;
      })`;
    
      return script;
    }
compile函数的用法如下。

    var parse = eval(compile(template));
    div.innerHTML = parse({ supplies: [ "broom", "mop", "cleaner" ] });
    //   <ul>
    //     <li>broom</li>
    //     <li>mop</li>
    //     <li>cleaner</li>
    //   </ul>

### 3.标签模板

模板字符串的功能，不仅仅是上面这些。它可以紧跟在一个函数名后面，该函数将被调用来处理这个模板字符串。这被称为“标签模板”功能（tagged template）。

    alert`123`
    // 等同于
    alert(123)

标签模板其实不是模板，而是函数调用的一种特殊形式。“标签”指的就是函数，紧跟在后面的模板字符串就是它的参数。

但是，如果模板字符里面有变量，就不是简单的调用了，而是会将模板字符串先处理成多个参数，再调用函数。

    var a = 5;
    var b = 10;
    
    tag`Hello ${ a + b } world ${ a * b }`;
    // 等同于
    tag(['Hello ', ' world ', ''], 15, 50);
    
上面代码中，模板字符串前面有一个标识名tag，它是一个函数。整个表达式的返回值，就是tag函数处理模板字符串后的返回值。

函数tag依次会接收到多个参数。

    function tag(stringArr, value1, value2){
      // ...
    }
    
    // 等同于
    
    function tag(stringArr, ...values){
      // ...
    }
    
tag函数的第一个参数是一个数组，该数组的成员是模板字符串中那些没有变量替换的部分，也就是说，变量替换只发生在数组的第一个成员与第二个成员之间、第二个成员与第三个成员之间，以此类推。

tag函数的其他参数，都是模板字符串各个变量被替换后的值。由于本例中，模板字符串含有两个变量，因此tag会接受到value1和value2两个参数。

tag函数所有参数的实际值如下。

第一个参数：['Hello ', ' world ', '']
第二个参数: 15
第三个参数：50
也就是说，tag函数实际上以下面的形式调用。

    tag(['Hello ', ' world ', ''], 15, 50)
    
我们可以按照需要编写tag函数的代码。下面是tag函数的一种写法，以及运行结果。

    var a = 5;
    var b = 10;
    
    function tag(s, v1, v2) {
      console.log(s[0]);
      console.log(s[1]);
      console.log(s[2]);
      console.log(v1);
      console.log(v2);
    
      return "OK";
    }
    
    tag`Hello ${ a + b } world ${ a * b}`;
    // "Hello "
    // " world "
    // ""
    // 15
    // 50
    // "OK"
    
下面是一个更复杂的例子。

    var total = 30;
    var msg = passthru`The total is ${total} (${total*1.05} with tax)`;
    
    function passthru(literals) {
      var result = '';
      var i = 0;
    
      while (i < literals.length) {
        result += literals[i++];
        if (i < arguments.length) {
          result += arguments[i];
        }
      }
    
      return result;
    }
    
    msg // "The total is 30 (31.5 with tax)"
    
上面这个例子展示了，如何将各个参数按照原来的位置拼合回去。

passthru函数采用rest参数的写法如下。
    
    function passthru(literals, ...values) {
      var output = "";
      for (var index = 0; index < values.length; index++) {
        output += literals[index] + values[index];
      }
    
      output += literals[index]
      return output;
    }
    
“标签模板”的一个重要应用，就是过滤HTML字符串，防止用户输入恶意内容。

    var message =
      SaferHTML`<p>${sender} has sent you a message.</p>`;
    
    function SaferHTML(templateData) {
      var s = templateData[0];
      for (var i = 1; i < arguments.length; i++) {
        var arg = String(arguments[i]);
    
        // Escape special characters in the substitution.
        s += arg.replace(/&/g, "&amp;")
                .replace(/</g, "&lt;")
                .replace(/>/g, "&gt;");
    
        // Don't escape special characters in the template.
        s += templateData[i];
      }
      return s;
    }
    
上面代码中，sender变量往往是用户提供的，经过SaferHTML函数处理，里面的特殊字符都会被转义。

    var sender = '<script>alert("abc")</script>'; // 恶意代码
    var message = SaferHTML`<p>${sender} has sent you a message.</p>`;
    
    message
    // <p>&lt;script&gt;alert("abc")&lt;/script&gt; has sent you a message.</p>

标签模板的另一个应用，就是多语言转换（国际化处理）。

    i18n`Welcome to ${siteName}, you are visitor number ${visitorNumber}!`
    // "欢迎访问xxx，您是第xxxx位访问者！"
    
模板字符串本身并不能取代Mustache之类的模板库，因为没有条件判断和循环处理功能，但是通过标签函数，你可以自己添加这些功能。

    // 下面的hashTemplate函数
    // 是一个自定义的模板处理函数
    var libraryHtml = hashTemplate`
      <ul>
        #for book in ${myBooks}
          <li><i>#{book.title}</i> by #{book.author}</li>
        #end
      </ul>
    `;
    
除此之外，你甚至可以使用标签模板，在JavaScript语言之中嵌入其他语言。

    jsx`
      <div>
        <input
          ref='input'
          onChange='${this.handleChange}'
          defaultValue='${this.state.value}' />
          ${this.state.value}
       </div>
    `
    
上面的代码通过jsx函数，将一个DOM字符串转为React对象。你可以在Github找到jsx函数的具体实现。

下面则是一个假想的例子，通过java函数，在JavaScript代码之中运行Java代码。
    
    java`
    class HelloWorldApp {
      public static void main(String[] args) {
        System.out.println(“Hello World!”); // Display the string.
      }
    }
    `
    HelloWorldApp.main();
    
模板处理函数的第一个参数（模板字符串数组），还有一个raw属性。
    
    console.log`123`
    // ["123", raw: Array[1]]
    
上面代码中，console.log接受的参数，实际上是一个数组。该数组有一个raw属性，保存的是转义后的原字符串。

请看下面的例子。
    
    tag`First line\nSecond line`
    
    function tag(strings) {
      console.log(strings.raw[0]);
      // "First line\\nSecond line"
    }
    
上面代码中，tag函数的第一个参数strings，有一个raw属性，也指向一个数组。该数组的成员与strings数组完全一致。比如，strings数组是["First line\nSecond line"]，那么strings.raw数组就是["First line\\nSecond line"]。两者唯一的区别，就是字符串里面的斜杠都被转义了。比如，strings.raw数组会将\n视为\\和n两个字符，而不是换行符。这是为了方便取得转义之前的原始模板而设计的。

## 函数  
### 1.rest参数  
ES6 引入 rest 参数（形式为“...变量名”），用于获取函数的多余参数，这样就不需要使用arguments对象了。rest 参数搭配的变量是一个数组，该变量将多余的参数放入数组中。

    function add(...values) {
      let sum = 0;
    
      for (var val of values) {
        sum += val;
      }
    
      return sum;
    }
    
    add(2, 5, 3) // 10
    
上面代码的add函数是一个求和函数，利用 rest 参数，可以向该函数传入任意数目的参数。

下面是一个 rest 参数代替arguments变量的例子。

    // arguments变量的写法
    function sortNumbers() {
      return Array.prototype.slice.call(arguments).sort();
    }
    
    // rest参数的写法
    const sortNumbers = (...numbers) => numbers.sort();
    
上面代码的两种写法，比较后可以发现，rest 参数的写法更自然也更简洁。

rest 参数中的变量代表一个数组，所以数组特有的方法都可以用于这个变量。下面是一个利用 rest 参数改写数组push方法的例子。

    function push(array, ...items) {
      items.forEach(function(item) {
        array.push(item);
        console.log(item);
      });
    }
    
    var a = [];
    push(a, 1, 2, 3)
注意，rest 参数之后不能再有其他参数（即只能是最后一个参数），否则会报错。
    
    // 报错
    function f(a, ...b, c) {
      // ...
    }
函数的length属性，不包括 rest 参数。
    
    (function(a) {}).length  // 1
    (function(...a) {}).length  // 0
    (function(a, ...b) {}).length  // 1
### 2.扩展运算符  
#### 含义  
扩展运算符（spread）是三个点（...）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列。
    
    console.log(...[1, 2, 3])
    // 1 2 3
    
    console.log(1, ...[2, 3, 4], 5)
    // 1 2 3 4 5
    
    [...document.querySelectorAll('div')]
    // [<div>, <div>, <div>]
    
该运算符主要用于函数调用。
    
    function push(array, ...items) {
      array.push(...items);
    }
    
    function add(x, y) {
      return x + y;
    }
    
    var numbers = [4, 38];
    add(...numbers) // 42
    
上面代码中，array.push(...items)和add(...numbers)这两行，都是函数的调用，它们的都使用了扩展运算符。该运算符将一个数组，变为参数序列。

扩展运算符与正常的函数参数可以结合使用，非常灵活。
    
    function f(v, w, x, y, z) { }
    var args = [0, 1];
    f(-1, ...args, 2, ...[3]);
    
#### 替代数组apply方法  
由于扩展运算符可以展开数组，所以不再需要apply方法，将数组转为函数的参数了。
    
    // ES5的写法
    function f(x, y, z) {
      // ...
    }
    var args = [0, 1, 2];
    f.apply(null, args);
    
    // ES6的写法
    function f(x, y, z) {
      // ...
    }
    var args = [0, 1, 2];
    f(...args);
    
下面是扩展运算符取代apply方法的一个实际的例子，应用Math.max方法，简化求出一个数组最大元素的写法。
    
    // ES5的写法
    Math.max.apply(null, [14, 3, 77])
    
    // ES6的写法
    Math.max(...[14, 3, 77])
    
    // 等同于
    Math.max(14, 3, 77);
    
上面代码表示，由于JavaScript不提供求数组最大元素的函数，所以只能套用Math.max函数，将数组转为一个参数序列，然后求最大值。有了扩展运算符以后，就可以直接用Math.max了。

另一个例子是通过push函数，将一个数组添加到另一个数组的尾部。
    
    // ES5的写法
    var arr1 = [0, 1, 2];
    var arr2 = [3, 4, 5];
    Array.prototype.push.apply(arr1, arr2);
    
    // ES6的写法
    var arr1 = [0, 1, 2];
    var arr2 = [3, 4, 5];
    arr1.push(...arr2);
    
上面代码的ES5写法中，push方法的参数不能是数组，所以只好通过apply方法变通使用push方法。有了扩展运算符，就可以直接将数组传入push方法。

下面是另外一个例子。
    
    // ES5
    new (Date.bind.apply(Date, [null, 2015, 1, 1]))
    // ES6
    new Date(...[2015, 1, 1]);    
    
### 3.箭头函数  
#### 基本用法  
ES6允许使用“箭头”（=>）定义函数。
    
    var f = v => v;
    
上面的箭头函数等同于：
    
    var f = function(v) {
      return v;
    };
    
如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。
    
    var f = () => 5;
    // 等同于
    var f = function () { return 5 };
    
    var sum = (num1, num2) => num1 + num2;
    // 等同于
    var sum = function(num1, num2) {
      return num1 + num2;
    };
    
如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用return语句返回。

    var sum = (num1, num2) => { return num1 + num2; }
    
由于大括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在对象外面加上括号。

    var getTempItem = id => ({ id: id, name: "Temp" });
    
箭头函数可以与变量解构结合使用。

    const full = ({ first, last }) => first + ' ' + last;
    
    // 等同于
    function full(person) {
      return person.first + ' ' + person.last;
    }
    
箭头函数使得表达更加简洁。
    
    const isEven = n => n % 2 == 0;
    const square = n => n * n;
    
上面代码只用了两行，就定义了两个简单的工具函数。如果不用箭头函数，可能就要占用多行，而且还不如现在这样写醒目。

箭头函数的一个用处是简化回调函数。
    
    // 正常函数写法
    [1,2,3].map(function (x) {
      return x * x;
    });
    
    // 箭头函数写法
    [1,2,3].map(x => x * x);
    
另一个例子是
    
    // 正常函数写法
    var result = values.sort(function (a, b) {
      return a - b;
    });
    
    // 箭头函数写法
    var result = values.sort((a, b) => a - b);
    
下面是rest参数与箭头函数结合的例子。
    
    const numbers = (...nums) => nums;
    
    numbers(1, 2, 3, 4, 5)
    // [1,2,3,4,5]
    
    const headAndTail = (head, ...tail) => [head, tail];
    
    headAndTail(1, 2, 3, 4, 5)
    // [1,[2,3,4,5]]
    
#### 使用注意点  
箭头函数有几个使用注意点。

（1）函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。

（2）不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。

（3）不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用Rest参数代替。

（4）不可以使用yield命令，因此箭头函数不能用作Generator函数。

上面四点中，第一点尤其值得注意。this对象的指向是可变的，但是在箭头函数中，它是固定的。

    function foo() {
      setTimeout(() => {
        console.log('id:', this.id);
      }, 100);
    }
    
    var id = 21;
    
    foo.call({ id: 42 });
    // id: 42
    
上面代码中，setTimeout的参数是一个箭头函数，这个箭头函数的定义生效是在foo函数生成时，而它的真正执行要等到100毫秒后。如果是普通函数，执行时this应该指向全局对象window，这时应该输出21。但是，箭头函数导致this总是指向函数定义生效时所在的对象（本例是{id: 42}），所以输出的是42。

箭头函数可以让setTimeout里面的this，绑定定义时所在的作用域，而不是指向运行时所在的作用域。下面是另一个例子。

    function Timer() {
      this.s1 = 0;
      this.s2 = 0;
      // 箭头函数
      setInterval(() => this.s1++, 1000);
      // 普通函数
      setInterval(function () {
        this.s2++;
      }, 1000);
    }
    
    var timer = new Timer();
    
    setTimeout(() => console.log('s1: ', timer.s1), 3100);
    setTimeout(() => console.log('s2: ', timer.s2), 3100);
    // s1: 3
    // s2: 0
    
上面代码中，Timer函数内部设置了两个定时器，分别使用了箭头函数和普通函数。前者的this绑定定义时所在的作用域（即Timer函数），后者的this指向运行时所在的作用域（即全局对象）。所以，3100毫秒之后，timer.s1被更新了3次，而timer.s2一次都没更新。

箭头函数可以让this指向固定化，这种特性很有利于封装回调函数。下面是一个例子，DOM事件的回调函数封装在一个对象里面。
    
    var handler = {
      id: '123456',
    
      init: function() {
        document.addEventListener('click',
          event => this.doSomething(event.type), false);
      },
    
      doSomething: function(type) {
        console.log('Handling ' + type  + ' for ' + this.id);
      }
    };
上面代码的init方法中，使用了箭头函数，这导致这个箭头函数里面的this，总是指向handler对象。否则，回调函数运行时，this.doSomething这一行会报错，因为此时this指向document对象。

this指向的固定化，并不是因为箭头函数内部有绑定this的机制，实际原因是箭头函数根本没有自己的this，导致内部的this就是外层代码块的this。正是因为它没有this，所以也就不能用作构造函数。

所以，箭头函数转成ES5的代码如下。
    
    // ES6
    function foo() {
      setTimeout(() => {
        console.log('id:', this.id);
      }, 100);
    }
    
    // ES5
    function foo() {
      var _this = this;
    
      setTimeout(function () {
        console.log('id:', _this.id);
      }, 100);
    }
上面代码中，转换后的ES5版本清楚地说明了，箭头函数里面根本没有自己的this，而是引用外层的this。

请问下面的代码之中有几个this？
    
    function foo() {
      return () => {
        return () => {
          return () => {
            console.log('id:', this.id);
          };
        };
      };
    }
    
    var f = foo.call({id: 1});
    
    var t1 = f.call({id: 2})()(); // id: 1
    var t2 = f().call({id: 3})(); // id: 1
    var t3 = f()().call({id: 4}); // id: 1
上面代码之中，只有一个this，就是函数foo的this，所以t1、t2、t3都输出同样的结果。因为所有的内层函数都是箭头函数，都没有自己的this，它们的this其实都是最外层foo函数的this。

除了this，以下三个变量在箭头函数之中也是不存在的，指向外层函数的对应变量：arguments、super、new.target。
    
    function foo() {
      setTimeout(() => {
        console.log('args:', arguments);
      }, 100);
    }
    
    foo(2, 4, 6, 8)
    // args: [2, 4, 6, 8]
    
上面代码中，箭头函数内部的变量arguments，其实是函数foo的arguments变量。

另外，由于箭头函数没有自己的this，所以当然也就不能用call()、apply()、bind()这些方法去改变this的指向。
    
    (function() {
      return [
        (() => this.x).bind({ x: 'inner' })()
      ];
    }).call({ x: 'outer' });
    // ['outer']
    
上面代码中，箭头函数没有自己的this，所以bind方法无效，内部的this指向外部的this。

长期以来，JavaScript语言的this对象一直是一个令人头痛的问题，在对象方法中使用this，必须非常小心。箭头函数”绑定”this，很大程度上解决了这个困扰。










    
[1]:https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements "MDN 语句和声明"