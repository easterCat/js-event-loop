[工具 ui.dev](https://ui.dev/javascript-visualizer/?)

[极客时间 - 浏览器工作原理与实践](https://time.geekbang.org/column/article/119046)

## 浏览器的多进程架构

- 浏览器进程.主要负责界面显示、用户交互、子进程管理,同时提供存储等功能.
- 渲染进程.核心任务是将 HTML、CSS 和 JavaScript 转换为用户可以与之交互的网页,排版引擎 Blink 和 JavaScript 引擎 V8 都是运行在该进程中,默认情况下,Chrome 会为每个 Tab 标签创建一个渲染进程.出于安全考虑,渲染进程都是运行在沙箱模式下.
- GPU 进程.其实,Chrome 刚开始发布的时候是没有 GPU 进程的.而 GPU 的使用初衷是为了实现 3D CSS 的效果,只是随后网页、Chrome 的 UI 界面都选择采用 GPU 来绘制,这使得 GPU 成为浏览器普遍的需求.最后,Chrome 在其多进程架构上也引入了 GPU 进程.
- 网络进程.主要负责页面的网络资源加载,之前是作为一个模块运行在浏览器进程里面的,直至最近才独立出来,成为一个单独的进程.
- 插件进程.主要是负责插件的运行,因插件易崩溃,所以需要通过插件进程来隔离,以保证插件进程崩溃不会对浏览器和页面造成影响.

![浏览器的多进程架构](https://static001.geekbang.org/resource/image/b6/fc/b61cab529fa31301bde290813b4587fc.png)

## JavaScript 代码是按顺序执行的吗？

```js
showName();
console.log(myname);
console.log(noname);
var myname = "极客";
function showName() {
  console.log("函数showName被执行");
}
```

第 1 行输出“函数 showName 被执行”,第 2 行输出“undefined”,第 3 行输入“Uncaught ReferenceError: noname is not defined”

- 在执行过程中,若使用了未声明的变量,那么 JavaScript 执行会报错.
- 在一个变量定义之前使用它,不会出错,但是该变量的值会为 undefined,而不是定义时的值.
- 在一个函数定义之前使用它,不会出错,且函数能正确执行.

### 变量提升（Hoisting）

#### 变量声明

首先,js 中的声明和赋值是由两部分组成,其中变量声明默认是 undefined

```js
var myname = "极客时间";

// 以上代码可以看做两部分
var myname = undefined; //声明部分
myname = "极客时间"; //赋值部分
```

![如图所示](https://static001.geekbang.org/resource/image/ec/3c/ec882f2d9deec26ce168b409f274533c.png)

#### 函数声明

函数声明分为两种,分别是直接函数声明 function name(){},或者匿名函数赋值 var name = function(){}

```js
function foo() {
  console.log("foo");
}

var bar = function () {
  console.log("bar");
};
```

![如图所示](https://static001.geekbang.org/resource/image/61/77/611c09ab995b9b608d9c0db193266777.png)

#### 什么是变量提升

所谓的变量提升,是指在 JavaScript 代码执行过程中,JavaScript 引擎把变量的声明部分和函数的声明部分提升到代码开头的“行为”.变量被提升后,会给变量设置默认值,这个默认值就是我们熟悉的 undefined.

```js
showName();
console.log(myname);
var myname = "极客";
function showName() {
  console.log("函数showName被执行");
}
```

[点击查看代码执行](<https://ui.dev/javascript-visualizer/?code=showName()%0Aconsole.log(myname)%0Avar%20myname%20=%20%22%E6%9E%81%E5%AE%A2%22%0Afunction%20showName()%20%7B%0A%20%20%20%20console.log(%27%E5%87%BD%E6%95%B0showName%E8%A2%AB%E6%89%A7%E8%A1%8C%27);%0A%7D>)

用代码模拟变量提升

```js
/*
 * 变量提升部分
 */
// 把变量 myname提升到开头,
// 同时给myname赋值为undefined
var myname = undefined;
// 把函数showName提升到开头
function showName() {
  console.log("showName被调用");
}

/*
 * 可执行代码部分
 */
showName();
console.log(myname);
// 去掉var声明部分,保留赋值语句
myname = "极客时间";
```

这样就很明显看出 myname 输出 undefined 的原因,等到 console 的时候,myname 实际还没有完成赋值,所以输出默认的 undefined

> 可以在定义之前使用变量或者函数的原因——函数和变量在执行之前都提升到了代码开头.

#### JavaScript 代码的执行流程

实际上变量和函数声明在代码里的位置是不会改变的,而且是在编译阶段被 JavaScript 引擎放入内存中.一段 JavaScript 代码在执行之前需要被 JavaScript 引擎编译,编译完成之后,才会进入执行阶段.

![](https://static001.geekbang.org/resource/image/64/1e/649c6e3b5509ffd40e13ce9c91b3d91e.png)

#### 编译阶段

```js
var myname = undefined;
function showName() {
  console.log("showName被调用");
}

showName();
console.log(myname);
myname = "极客时间";
```

![](https://static001.geekbang.org/resource/image/06/13/0655d18ec347a95dfbf843969a921a13.png)

经过编译后,会生成两部分内容：执行上下文（Execution context）和可执行代码.执行上下文是 JavaScript 执行一段代码时的运行环境,比如调用一个函数,就会进入这个函数的执行上下文,确定该函数在执行期间用到的诸如 this、变量、对象以及函数等.在执行上下文中存在一个变量环境的对象（Viriable Environment）,该对象中保存了变量提升的内容.

```js
// 变量环境大致结构
VariableEnvironment:
     myname -> undefined,
     showName -> function : {console.log(myname)
```

##### 如何生成变量环境对象

```js
showName();
console.log(myname);
var myname = "极客时间";
function showName() {
  console.log("函数showName被执行");
}
```

- 第 1 行和第 2 行,由于这两行代码不是声明操作,所以 JavaScript 引擎不会做任何处理；
- 第 3 行,由于这行是经过 var 声明的,因此 JavaScript 引擎将在环境对象中创建一个名为 myname 的属性,并使用 undefined 对其初始化；
- 第 4 行,JavaScript 引擎发现了一个通过 function 定义的函数,所以它将函数定义存储到堆 (HEAP）中,并在环境对象中创建一个 showName 的属性,然后将该属性值指向堆中函数的位置

这样就生成了变量环境对象.接下来 JavaScript 引擎会把声明以外的代码编译为字节码.

#### 执行阶段

- 当执行到 showName 函数时,JavaScript 引擎便开始在变量环境对象中查找该函数,由于变量环境对象中存在该函数的引用,所以 JavaScript 引擎便开始执行该函数,并输出“函数 showName 被执行”结果.
- 接下来打印“myname”信息,JavaScript 引擎继续在变量环境对象中查找该对象,由于变量环境存在 myname 变量,并且其值为 undefined,所以这时候就输出 undefined.
- 接下来执行第 3 行,把“极客时间”赋给 myname 变量,赋值后变量环境中的 myname 属性值改变为“极客时间”

#### 代码中出现相同的变量或者函数

1. 如果是同名的函数,JavaScript 编译阶段会选择最后声明的那个.
2. 如果变量和函数同名,那么在编译阶段,变量的声明会被忽略

```js
function showName() {
  console.log("极客邦");
}
showName();
function showName() {
  console.log("极客时间");
}
showName();
```

[点击执行代码](https://ui.dev/javascript-visualizer/?code=%0Afunction%20showName%28%29%20%7B%0A%20%20%20%20console.log%28%27%E6%9E%81%E5%AE%A2%E9%82%A6%27%29%3B%0A%7D%0AshowName%28%29%3B%0Afunction%20showName%28%29%20%7B%0A%20%20%20%20console.log%28%27%E6%9E%81%E5%AE%A2%E6%97%B6%E9%97%B4%27%29%3B%0A%7D%0AshowName%28%29%3B%20)

- 首先是编译阶段.遇到了第一个 showName 函数,会将该函数体存放到变量环境中.接下来是第二个 showName 函数,继续存放至变量环境中,但是变量环境中已经存在一个 showName 函数了,此时,第二个 showName 函数会将第一个 showName 函数覆盖掉.这样变量环境中就只存在第二个 showName 函数了.
- 接下来是执行阶段.先执行第一个 showName 函数,但由于是从变量环境中查找 showName 函数,而变量环境中只保存了第二个 showName 函数,所以最终调用的是第二个函数,打印的内容是“极客时间”.第二次执行 showName 函数也是走同样的流程,所以输出的结果也是“极客时间”.

一段代码如果定义了两个相同名字的函数,那么最终生效的是最后一个函数.

```js
showName();
function showName() {
  console.log("极客时间");
}
var showName = "极客";
console.log(showName);
```

（该代码在 ui.dev 中执行结果不正确，ui.dev 没有实现编译阶段函数声明优先级的问题）

- 首先是编译阶段，遇到第一个函数声明 showName，然后将该函数体添加到变量环境，然后遇到第二个变量声明 showName，但是变量环境中已经存在一个 showName 函数，所以忽略该变量声明。
- 接下来进去执行阶段，先输出 showName 函数，之后完成 showName 赋值，再输出"极客"

> 函数声明提升要比变量声明提升的优先级要高一些，且不会被变量声明覆盖，但是会被变量赋值之后覆盖。

## 为什么 JavaScript 代码会出现栈溢出？

JavaScript 在遇到下面三种情况，会在代码执行之前进行编译并创建执行上下文

- 当 JavaScript 执行全局代码的时候，会编译全局代码并创建全局执行上下文，而且在整个页面的生存周期内，全局执行上下文只有一份。
- 当调用一个函数的时候，函数体内的代码会被编译，并创建函数执行上下文，一般情况下，函数执行结束之后，创建的函数执行上下文会被销毁。
- 当使用 eval 函数的时候，eval 的代码也会被编译，并创建执行上下文

#### 函数调用

函数调用就是运行一个函数

```js
var a = 1;
function add(x) {
  var b = 2;
  return x + b;
}
add(a);
```

这就是一个简单的函数声明 + 函数调用

在执行一段代码之前，js 引擎会先创建全局执行上下文，包含了声明的函数和变量

![](https://static001.geekbang.org/resource/image/7f/da/7fa2ed18e702861890d767ea547533da.png)

add 的函数调用过程

- 首先，创建全局执行上下文，执行代码，add 函数调用
- 对 add 函数进行编译，创建出改函数执行上下文和可执行代码
- 执行代码

![](https://static001.geekbang.org/resource/image/53/ca/537efd9e96771dc50737117e615533ca.png)

#### 栈

栈是一种数据结构，特点是后进先出

![](https://static001.geekbang.org/resource/image/5e/05/5e2bb65019053abfd5e7710e41d1b405.png)

#### 调用栈

当执行上下文创建之后，JavaScript 引擎会将执行上下文压入栈中，，通常把这种用来管理执行上下文的栈称为执行上下文栈，又称调用栈。

```js
var a = 2;
function add(b, c) {
  return b + c;
}
function addAll(b, c) {
  var d = 10;
  result = add(b, c);
  return a + result + d;
}
addAll(3, 6);
```

[点击执行代码](https://ui.dev/javascript-visualizer/?code=var%20a%20%3D%202%0Afunction%20add%28b%2Cc%29%7B%0A%20%20return%20b%2Bc%0A%7D%0Afunction%20addAll%28b%2Cc%29%7B%0Avar%20d%20%3D%2010%0Aresult%20%3D%20add%28b%2Cc%29%0Areturn%20%20a%2Bresult%2Bd%0A%7D%0AaddAll%283%2C6%29)

第一步，创建全局上下文，并将其压入栈底。
![](https://static001.geekbang.org/resource/image/a5/1d/a5d7ec1f8f296412acc045835b85431d.png)

第二步是调用 addAll 函数。
![](https://static001.geekbang.org/resource/image/7d/52/7d6c4c45db4ef9b900678092e6c53652.png)

第三步，当执行到 add 函数
![](https://static001.geekbang.org/resource/image/cc/37/ccfe41d906040031a7df1e4f1bce5837.png)

当 add 函数返回时，该函数的执行上下文就会从栈顶弹出
![](https://static001.geekbang.org/resource/image/03/96/03ca801a5372f941bf17d6088fee0f96.png)

addAll 执行最后一个相加操作后并返回，addAll 的执行上下文也会从栈顶部弹出，
![](https://static001.geekbang.org/resource/image/d0/7b/d0ac1d6e77735338fa97cc9a3f6c717b.png)

#### 栈溢出（Stack Overflow）

调用栈是有大小，当超出这个额度，会直接抛出错误

```js
function division(a, b) {
  return division(a, b);
}
console.log(division(1, 2));
```

执行代码，控制台会报错 超过了最大栈调用大小（Maximum call stack size exceeded）。

#### 获取调用栈深度

```js
var computeMaxCallStackSize = (function () {
  return function () {
    var size = 0;
    function cs() {
      try {
        size++;
        return cs();
      } catch (e) {
        return size + 1;
      }
    }
    return cs();
  };
})();
```

测试出的结果在 chrome 中是 12525

## var 缺陷以及为什么要引入 let 和 const？

### 作用域（scope）

作用域是指在程序中定义变量的区域，该位置决定了变量的生命周期。通俗地理解，作用域就是变量与函数的可访问范围，即作用域控制着变量和函数的可见性和生命周期。

在 ES6 之前，ES 的作用域只有两种：

- 全局作用域和函数作用域。全局作用域中的对象在代码中的任何地方都能访问，其生命周期伴随着页面的生命周期。
- 函数作用域就是在函数内部定义的变量或者函数，并且定义的变量或者函数只能在函数内部被访问。函数执行结束之后，函数内部定义的变量会被销毁。

#### 块级作用域

块级作用域就是使用一对大括号包裹的一段代码，比如函数、判断语句、循环语句，甚至单独的一个{}都可以被看作是一个块级作用域。如果一种语言支持块级作用域，那么其代码块内部定义的变量在代码块外部是访问不到的，并且等该代码块中的代码执行完成之后，代码块中定义的变量会被销毁。

```js
//if块
if (1) {
}

//while块
while (1) {}

//函数块
function foo() {}

//for循环块
for (let i = 0; i < 100; i++) {}

//单独一个块
{
}
```

#### 变量提升所带来的问题

- 变量容易在不被察觉的情况下被覆盖掉

```js
var myname = "极客时间";
function showName() {
  console.log(myname);
  if (0) {
    var myname = "极客邦";
  }
  console.log(myname);
}
showName();
```

- 本应销毁的变量没有被销毁

```js
function foo() {
  for (var i = 0; i < 7; i++) {}
  console.log(i);
}
foo();
```

#### ES6 是如何解决变量提升带来的缺陷

ES6 引入了 let 和 const 关键字，从而使 JavaScript 也能像其他语言一样拥有了块级作用域。

```js
function varTest() {
  var x = 1;
  if (true) {
    var x = 2; // 同样的变量!
    console.log(x); // 2
  }
  console.log(x); // 2
}
```

把 var 关键字替换为 let 关键字，让其支持块级作用域。

```js
function letTest() {
  let x = 1;
  if (true) {
    let x = 2; // 不同的变量
    console.log(x); // 2
  }
  console.log(x); // 1
}
```

let 关键字是支持块级作用域的，所以在编译阶段，JavaScript 引擎并不会把 if 块中通过 let 声明的变量存放到变量环境中，这也就意味着在 if 块通过 let 声明的关键字，并不会提升到全函数可见。

#### JavaScript 是如何支持块级作用域的

```js
function foo() {
  var a = 1;
  let b = 2;
  {
    let b = 3;
    var c = 4;
    let d = 5;
    console.log(a); //1
    console.log(b); //3
  }
  console.log(b); // 2
  console.log(c); //4
  console.log(d); // d is not defined
}
foo();
```

上面这段代码的执行流程:

- 第一步是编译并创建执行上下文

![](https://static001.geekbang.org/resource/image/f9/67/f9f67f2f53437218baef9dc724bd4c67.png)

1. 函数内部通过 var 声明的变量，在编译阶段全都被存放到变量环境里面了。
2. 通过 let 声明的变量，在编译阶段会被存放到词法环境（Lexical Environment）中。
3. 函数的作用域块内部，通过 let 声明的变量并没有被存放到词法环境中。

- 第二步继续执行代码

![](https://static001.geekbang.org/resource/image/7e/fa/7e0f7bc362e0dea21d27dc5fb08d06fa.png)

1.  函数的作用域块中通过 let 声明的变量，会被存放在词法环境的一个单独的区域中
2.  这个区域中的变量并不影响作用域块外面的变量

> 在词法环境内部，维护了一个小型栈结构，栈底是函数最外层的变量，进入一个作用域块后，就会把该作用域块内部的变量压到栈顶；当作用域执行完成之后，该作用域的信息就会从栈顶弹出，这就是词法环境的结构。变量是指通过 let 或者 const 声明的变量。

- 执行到 console.log(a)

就需要在词法环境和变量环境中查找变量 a 的值了，具体查找方式是：沿着词法环境的栈顶向下查询，如果在词法环境中的某个块中查找到了，就直接返回给 JavaScript 引擎，如果没有查找到，那么继续在变量环境中查找。

![](https://static001.geekbang.org/resource/image/06/08/06c06a756632acb12aa97b3be57bb908.png)

- 当作用域块执行结束之后，其内部定义的变量就会从词法环境的栈顶弹出

![](https://static001.geekbang.org/resource/image/d4/28/d4f99640d62feba4202aa072f6369d28.png)

块级作用域就是通过词法环境的栈结构来实现的，而变量提升是通过变量环境来实现，通过这两者的结合，JavaScript 引擎也就同时支持了变量提升和块级作用域了。

#### 暂时性死区

执行函数时才有进行编译，抽象语法树(AST）在进入函数阶段就生成了，并且函数内部作用域是已经明确了，所以进入块级作用域不会有编译过程，只不过通过 let 或者 const 声明的变量会在进入块级作用域的时被创建，但是在该变量没有赋值之前，引用该变量 JavaScript 引擎会抛出错误---这就是“暂时性死区”

```js
let myname = "极客时间";
{
  console.log(myname);
  let myname = "极客邦";
}
```

控制台输出结果 Uncaught ReferenceError: Cannot access 'myname' before initialization

> 在块作用域内，let 声明的变量被提升，但变量只是创建被提升，初始化并没有被提升，在初始化之前使用变量，就会形成一个暂时性死区。

- var 的创建和初始化被提升，赋值不会被提升。
- let 的创建被提升，初始化和赋值不会被提升。
- function 的创建、初始化和赋值均会被提升。

## 代码中出现相同的变量，JavaScript 引擎是如何选择的？

### 作用域链和闭包

ES6 是通过变量环境和词法环境来同时支持变量提升和块级作用域，通过词法环境和变量环境来查找变量就涉及到作用域链的概念。

```js
function bar() {
  console.log(myName);
}
function foo() {
  var myName = "极客邦";
  bar();
}
var myName = "极客时间";
foo();
```

![](https://static001.geekbang.org/resource/image/87/f7/87d8bbc2bb62b03131802fba074146f7.png)

看到调用栈的时候，可能第一时间觉得变量查找是按照栈的顺序走，但是 js 的作用域是静态作用域，在编译的时候已经确定了，不会变动。

#### 作用域链

其实在每个执行上下文的变量环境中，都包含了一个外部引用，用来指向外部的执行上下文，我们把这个外部引用称为 outer。

当一段代码使用了一个变量时，JavaScript 引擎首先会在“当前的执行上下文”中查找该变量，比如上面那段代码在查找 myName 变量时，如果在当前的变量环境中没有查找到，那么 JavaScript 引擎会继续在 outer 所指向的执行上下文中查找。

![](https://static001.geekbang.org/resource/image/20/a7/20a832656434264db47c93e657e346a7.png)

从图中可以看到，bar 函数和 foo 函数的 outer 都是指向全局上下文的，这也就意味着如果在 bar 函数或者 foo 函数中使用了外部变量，那么 JavaScript 引擎会去全局执行上下文中查找。而 bar=》全局这个链条就是作用域链。

#### 词法作用域

上面代码中，foo 函数调用的 bar 函数，但为什么 bar 函数的 outer 是全局上下文，这是因为词法作用域的原因。

词法作用域就是指作用域是由代码中函数声明的位置来决定的，所以词法作用域是静态的作用域，通过它就能够预测代码在执行过程中如何查找标识符。

![](https://static001.geekbang.org/resource/image/21/39/216433d2d0c64149a731d84ba1a07739.png)

从图中可以看出，词法作用域就是根据代码的位置来决定的，其中 main 函数包含了 bar 函数，bar 函数中包含了 foo 函数，因为 JavaScript 作用域链是由词法作用域决定的，所以整个词法作用域链的顺序是：foo 函数作用域—>bar 函数作用域—>main 函数作用域—> 全局作用域。

上文 bar 函数的 outer 为什么是全局上下文，这是因为根据词法作用域，foo 和 bar 的上级作用域都是全局作用域，所以如果 foo 或者 bar 函数使用了一个它们没有定义的变量，那么它们会到全局作用域去查找。也就是说，词法作用域是代码编译阶段就决定好的，和函数是怎么调用的没有关系。

#### 块级作用域中的变量查找

```js
function bar() {
  var myName = "极客世界";
  let test1 = 100;
  if (1) {
    let myName = "Chrome浏览器";
    console.log(test); // 1
  }
}
function foo() {
  var myName = "极客邦";
  let test = 2;
  {
    let test = 3;
    bar();
  }
}
var myName = "极客时间";
let myAge = 10;
let test = 1;
foo();
```

![](https://static001.geekbang.org/resource/image/25/a7/25053af5ae30c8be991fa14631cde0a7.png)

首先是编译阶段，生成执行上下文。直到执行 bar，此时会在当前函数执行上下文中去查找，但是 bar 函数的执行上下文没有定义 test，根据词法作用域规则，会在 bar 函数的 outer 作用域的找变量 test，输出 1.

### 闭包

```js
function foo() {
  var myName = "极客时间";
  let test1 = 1;
  const test2 = 2;
  var innerBar = {
    getName: function () {
      console.log(test1);
      return myName;
    },
    setName: function (newName) {
      myName = newName;
    },
  };
  return innerBar;
}
var bar = foo();
bar.setName("极客邦");
bar.getName();
console.log(bar.getName());
```

![](https://static001.geekbang.org/resource/image/d5/ef/d5587b76427a56c5f0b0571e4264b7ef.png)

根据词法作用域的规则，内部函数 getName 和 setName 总是可以访问它们的外部函数 foo 中的变量，所以当 innerBar 对象返回给全局变量 bar 时，虽然 foo 函数已经执行结束，但是 getName 和 setName 函数依然可以使用 foo 函数中的变量 myName 和 test1。

![](https://static001.geekbang.org/resource/image/ee/3f/ee7c1ca481875ad4bdeb4383bd1f883f.png)

foo 函数执行完成之后，其执行上下文从栈顶弹出了，但是由于返回的 setName 和 getName 方法中使用了 foo 函数内部的变量 myName 和 test1，所以这两个变量依然保存在内存中。

因为除了 setName 和 getName 函数之外，其他任何地方都是无法访问的，我们就可以把这个称为 foo 函数的闭包。

> 在 JavaScript 中，根据词法作用域的规则，内部函数总是可以访问其外部函数中声明的变量，当通过调用一个外部函数返回一个内部函数后，即使该外部函数已经执行结束了，但是内部函数引用外部函数的变量依然保存在内存中，我们就把这些变量的集合称为闭包。比如外部函数是 foo，那么这些变量的集合就称为 foo 函数的闭包。

#### 闭包是如何使用呢?

当执行到 bar.setName 方法中的 myName = "极客邦"这句代码时，JavaScript 引擎会沿着“当前执行上下文–>foo 函数闭包–> 全局执行上下文”的顺序来查找 myName 变量

![](https://static001.geekbang.org/resource/image/50/46/50e4ba60fc7e420e83b35b95e379b246.png)

也可以通过“开发者工具”来看看闭包的情况

```js
setName: function (newName) {
                debugger;
                myName = newName;
            },
```

可以看到在 scope 中，“Local–>Closure(foo)–>Global”就是一个完整的作用域链。

#### 闭包是怎么回收的

通常，如果引用闭包的函数是一个全局变量，那么闭包会一直存在直到页面关闭；但如果这个闭包以后不再使用的话，就会造成内存泄漏。如果引用闭包的函数是个局部变量，等函数销毁后，在下次 JavaScript 引擎执行垃圾回收时，判断闭包这块内容如果已经不再被使用了，那么 JavaScript 引擎的垃圾回收器就会回收这块内存。

> 如果该闭包会一直使用，那么它可以作为全局变量而存在；但如果使用频率不高，而且占用内存又比较大的话，那就尽量让它成为一个局部变量。

## 从 JavaScript 执行上下文的视角讲清楚 this

![](https://static001.geekbang.org/resource/image/b3/8d/b398610fd8060b381d33afc9b86f988d.png)

### this 是什么

this 是和执行上下文绑定的，也就是说每个执行上下文中都有一个 this。执行上下文主要分为三种——全局执行上下文、函数执行上下文和 eval 执行上下文，所以对应的 this 也只有这三种——全局执行上下文中的 this、函数中的 this 和 eval 中的 this。

#### 全局执行上下文中的 this

全局执行上下文中的 this 是指向 window 对象的。这也是 this 和作用域链的唯一交点，作用域链的最底端包含了 window 对象，全局执行上下文中的 this 也是指向 window 对象。

#### 函数执行上下文中的 this

```js
function foo() {
  console.log(this); // window
}
foo();
```

在默认情况下调用一个函数，其执行上下文中的 this 也是指向 window 对象的。

#### 设置函数执行上下文中的 this 值

1. 通过函数的 call 方法设置

可以通过函数的 call 方法来设置函数执行上下文的 this 指向

```**js**
let bar = {
    myName: "极客邦",
    test1: 1,
};
function foo() {
    console.log("this :>> ", this);
    this.myName = "极客时间";
}
// foo(); // this => window
foo.call(bar); // this => bar
// foo.apply(bar); // this => bar
// foo.bind(bar); // this => bar
console.log(bar);
console.log(myName);
```

除了 call 方法，你还可以使用 bind 和 apply 方法来设置函数执行上下文中的 this.

2. 通过对象调用方法设置

```js
var myObj = {
  name: "极客时间",
  showThis: function () {
    console.log(this); // myObj
  },
};
myObj.showThis(); // 按照 myObj.showThis.call(myObj) 理解
```

使用对象来调用其内部的一个方法，该方法的 this 是指向对象本身的。

**将上面代码更改为赋值的方式**

```js
var myObj = {
  name: "极客时间",
  showThis: function () {
    this.name = "极客邦";
    console.log(this); // 全局 window 对象
  },
};
var foo = myObj.showThis;
foo();
```

> 在全局环境中调用一个函数，函数内部的 this 指向的是全局变量 window。通过一个对象来调用其内部的一个方法，该方法的执行上下文中的 this 指向对象本身。

3. 通过构造函数中设置

```js
function CreateObj() {
  this.name = "极客时间";
}
var myObj = new CreateObj();
```

当执行 new CreateObj() 的时候，JavaScript 引擎做了如下四件事：

- 首先创建了一个空对象 tempObj；
- 接着调用 CreateObj.call 方法，并将 tempObj 作为 call 方法的参数，这样当 CreateObj 的执行上下文创建时，它的 this 就指向了 tempObj 对象；
- 然后执行 CreateObj 函数，此时的 CreateObj 函数执行上下文中的 this 指向了 tempObj 对象；
- 最后返回 tempObj 对象。

通过 new 关键字构建好一个新对象，并且构造函数中的 this 其实就是新对象本身。

#### this 的设计缺陷以及应对方案

1. 嵌套函数中的 this 不会从外层函数中继承

```js
var myObj = {
  name: "极客时间",
  showThis: function () {
    console.log(this);
    function bar() {
      console.log(this);
    }
    bar();
  },
};
myObj.showThis();
```

函数 bar 中的 this 指向的是全局 window 对象，而函数 showThis 中的 this 指向的是 myObj 对象。this 没有作用域的限制，这点和变量不一样，所以嵌套函数不会从调用它的函数中继承 this，这样会造成很多不符合直觉的代码。

```js
var myObj = {
  name: "极客时间",
  showThis: function () {
    console.log(this);
    var self = this;
    function bar() {
      self.name = "极客邦";
    }
    bar();
  },
};
myObj.showThis();
console.log(myObj.name);
console.log(window.name);
```

解决 1，在 showThis 函数中声明一个变量 self 用来保存 this，然后在 bar 函数中使用 self。这个方法的的本质是把 this 体系转换为了作用域的体系。

```js
var myObj = {
  name: "极客时间",
  showThis: function () {
    console.log(this);
    var bar = () => {
      this.name = "极客邦";
      console.log(this);
    };
    bar();
  },
};
myObj.showThis();
console.log(myObj.name);
console.log(window.name);
```

解决 2，你也可以使用 ES6 中的箭头函数来解决这个问题。因为 ES6 中的箭头函数并不会创建其自身的执行上下文，所以箭头函数中的 this 取决于它的外部函数。

2. 普通函数中的 this 默认指向全局对象 window

在默认情况下调用一个函数，其执行上下文中的 this 是默认指向全局对象 window 的。因为在实际工作中，我们并不希望函数执行上下文中的 this 默认指向全局对象，因为这样会打破数据的边界，造成一些误操作。如果要让函数执行上下文中的 this 指向某个对象，最好的方式是通过 call 方法来显示调用。可以通过设置 JavaScript 的“严格模式”来解决。在严格模式下，默认执行一个函数，其函数的执行上下文中的 this 值是 undefined，这就解决上面的问题了。

## 浏览器的页面循环系统

### WebAPI：setTimeout 是如何实现的？

setTimeout 是一个定时器,用来指定某个函数在多少毫秒之后执行.

#### 浏览器怎么实现 setTimeout

渲染进程中所有运行在主线程上的任务都需要先添加到消息队列,然后事件循环系统再按照顺序执行消息队列中的任务.

典型的事件：

- 当接收到 HTML 文档数据,渲染引擎就会将“解析 DOM”事件添加到消息队列中,
- 当用户改变了 Web 页面的窗口大小,渲染引擎就会将“重新布局”的事件添加到消息队列中.
- 当触发了 JavaScript 引擎垃圾回收机制,渲染引擎会将“垃圾回收”任务添加到消息队列中.
- 同样,如果要执行一段异步 JavaScript 代码,也是需要将执行任务添加到消息队列中.

为了支持定时器的实现,浏览器增加了延时队列.

> 在 Chrome 中除了正常使用的消息队列之外,还有另外一个消息队列,这个队列中维护了需要延迟执行的任务列表,包括了定时器和 Chromium 内部一些需要延迟执行的任务.所以当通过 JavaScript 创建一个定时器时,渲染进程会将该定时器的回调任务添加到延迟队列中.

> 延迟队列其实是一个 hashmap 结构,等到执行这个结构的时候,会计算 hashmap 中的每个任务是否到期了,到期了就去执行,直到所有到期的任务都执行结束,才会进入下一轮循环！

#### 使用 setTimeout 的一些注意事项

##### 如果当前任务执行时间过久,会影响定时器任务的执行

```js
console.time("Array initialize");
let num = 0;
for (let index = 0; index < 9999999; index++) {
  num = num + index;
}
console.log("num :>> ", num);
setTimeout(() => {
  console.log("setTimeout 0s");
  console.timeEnd("Array initialize");
}, 0);
```

输出结果是--setTimeout 0s ; 定时器任务的执行时间.html:21 Array initialize: 135.243896484375 ms--,原因在于 setTimeout 会加入任务到延迟消息队列,当主线程的任务全部清空,会从延迟队列中把任务依次执行.但是主线程的任务执行也有时间消耗,遇到高耗时任务的时候,setTimeout 无法精确控制延时执行时间,与预想时间会有差异.

##### 如果 setTimeout 存在嵌套调用,那么系统会设置最短时间间隔为 4 毫秒

```js
var timer = setTimeout(function fn() {
  var timer2 = setTimeout(fn, 1);
  console.log(new Date().getSeconds());
}, 1);
```

##### 未激活的页面,setTimeout 执行最小间隔是 1000 毫秒

```js
var timer = setTimeout(function fn() {
  var timer2 = setTimeout(fn, 100);
  console.log(new Date().getSeconds());
}, 100);
```

在浏览器中打开,观看控制台打印出来的秒数,每秒打印 9-10 次,因任务执行时间不准确会有偏差.之后,要么随意打开一个新 tab 切换过去,要么切换到音乐播放器听会歌曲,然后在切换回来看浏览器控制台,会发现秒数打印是一秒一次了.

> setInterval 会得到同样效果

##### 延时执行时间有最大值 2147483647 毫秒

Chrome、Safari、Firefox 都是以 32 个 bit 来存储延时值的,32bit 最大只能存放的数字是 2147483647 毫秒,这就意味着,如果 setTimeout 设置的延迟值大于 2147483647 毫秒（大约 24.8 天）时就会溢出,那么相当于延时值被设置为 0 了,这导致定时器会被立即执行.

```js
function fn() {
  setTimeout(() => {
    console.log("go");
  }, 2147483647);
}

function fn2() {
  setTimeout(() => {
    console.log("go2");
  }, 2147483648);
}

fn();
fn2();
```

##### 使用 setTimeout 设置的回调函数中的 this 不符合直觉

如果被 setTimeout 推迟执行的回调函数是某个对象的方法,那么该方法中的 this 关键字将指向全局环境,而不是定义时所在的那个对象.

```js
var name = 1;
var MyObj = {
  name: 2,
  showName: function () {
    console.log(this.name);
  },
};
setTimeout(MyObj.showName, 1000);
```

这里输出的是 1,因为这段代码在编译的时候,执行上下文中的 this 会被设置为全局 window,如果是严格模式,会被设置为 undefined.

> 可以通过匿名函数或者 bind 来解决

##### setTimeout 执行时间无法精确

```js
setTimeout(() => {
  console.log(2);
  setTimeout(() => {
    console.log(3);
  }, 99);
}, 100);
setTimeout(() => {
  console.log(1);
}, 200);
```

在浏览器中多刷新几次,然后看看控制台中打印的内容,并不一定就是输出预想的 2,3,1

#### requestAnimationFrame

使用 requestAnimationFrame 不需要设置具体的时间,由系统来决定回调函数的执行时间,requestAnimationFrame 里面的回调函数是在页面刷新之前执行,它跟着屏幕的刷新频率走,保证每个刷新间隔只执行一次,内如果页面未激活的话,requestAnimationFrame 也会停止渲染,这样既可以保证页面的流畅性,又能节省主线程执行函数的开销

### WebAPI：XMLHttpRequest 是怎么实现的？

### 宏任务和微任务

- macro-task（宏任务）大概包括：script(整体代码), setTimeout, setInterval, setImmediate（NodeJs）, I/O, UI rendering.
- micro-task（微任务）大概包括: process.nextTick（NodeJs）, Promise, Object.observe(已废弃), MutationObserver(html5 新特性)

#### 在主线程上执行的宏任务:

- 渲染事件（如解析 DOM、计算布局、绘制）；
- 用户交互事件（如鼠标点击、滚动页面、放大缩小等）；
- JavaScript 脚本执行事件；
- 网络请求完成、文件读写完成事件.

> 为了协调这些任务有条不紊地在主线程上执行,页面进程引入了消息队列和事件循环机制,渲染进程内部会维护多个消息队列,比如延迟执行队列和普通的消息队列.然后主线程采用一个 for 循环,不断地从这些任务队列中取出任务并执行任务.我们把这些消息队列中的任务称为宏任务.

#### 宏任务的执行过程:

- 先从多个消息队列中选出一个最老的任务,这个任务称为 oldestTask；
- 然后循环系统记录任务开始执行的时间,并把这个 oldestTask 设置为当前正在执行的任务；
- 当任务执行完成之后,删除当前正在执行的任务,并从对应的消息队列中删除掉这个 oldestTask；
- 最后统计执行完成的时长等信息.

#### 为什么宏任务难以满足对时间精度要求较高的任务?

页面的渲染事件、各种 IO 的完成事件、执行 JavaScript 脚本的事件、用户交互的事件等都随时有可能被添加到消息队列中,而且添加事件是由系统操作的,JavaScript 代码不能准确掌控任务要添加到队列中的位置,控制不了任务在消息队列中的位置,所以很难控制开始执行任务的时间.

#### 微任务系统是怎么运转起来的？

当 JavaScript 执行一段脚本的时候,V8 会为其创建一个全局执行上下文,在创建全局执行上下文的同时,V8 引擎也会在内部创建一个微任务队列.顾名思义,这个微任务队列就是用来存放微任务的,因为在当前宏任务执行的过程中,有时候会产生多个微任务,这时候就需要使用这个微任务队列来保存这些微任务了.不过这个微任务队列是给 V8 引擎内部使用的,所以你是无法通过 JavaScript 直接访问的.

> 也就是说每个宏任务都关联了一个微任务队列.

#### 产生微任务有两种方式

- 第一种方式是使用 MutationObserver 监控某个 DOM 节点,然后再通过 JavaScript 来修改这个节点,或者为这个节点添加、删除部分子节点,当 DOM 节点发生变化时,就会产生 DOM 变化记录的微任务.
- 第二种方式是使用 Promise,当调用 Promise.resolve() 或者 Promise.reject() 的时候,也会产生微任务.

#### 微任务队列是何时被执行的

通常情况下,在当前宏任务中的 JavaScript 快执行完成时,也就在 JavaScript 引擎准备退出全局执行上下文并清空调用栈的时候,JavaScript 引擎会检查全局执行上下文中的微任务队列,然后按照顺序执行队列中的微任务.WHATWG 把执行微任务的时间点称为检查点.

如果在执行微任务的过程中,产生了新的微任务,同样会将该微任务添加到微任务队列中,V8 引擎一直循环执行微任务队列中的任务,直到队列为空才算执行结束.也就是说在执行微任务过程中产生的新的微任务并不会推迟到下个宏任务中执行,而是在当前的宏任务中继续执行.

#### 微任务队列总结

- 微任务和宏任务是绑定的,每个宏任务在执行时,会创建自己的微任务队列.
- 微任务的执行时长会影响到当前宏任务的时长.比如一个宏任务在执行过程中,产生了 100 个微任务,执行每个微任务的时间是 10 毫秒,那么执行这 100 个微任务的时间就是 1000 毫秒,也可以说这 100 个微任务让宏任务的执行时间延长了 1000 毫秒.所以你在写代码的时候一定要注意控制微任务的执行时长.
- 在一个宏任务中,分别创建一个用于回调的宏任务和微任务,无论什么情况下,微任务都早于宏任务执行.
