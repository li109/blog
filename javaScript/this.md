# JavaScript夯实基础系列（三）：this
&emsp;&emsp;在JavaScript中，函数的每次调用都会拥有一个**执行上下文**，通过this关键字指向该上下文。函数中的代码在函数定义时不会执行，只有在函数被调用时才执行。函数调用的方式有四种：*作为函数调用*、*作为方法调用*、*作为构造函数调用*以及*间接调用*，判定this指向的规则跟函数调用的方式有关。<br/>
### 一、作为函数的调用
&emsp;&emsp;**作为函数调用**是指函数独立执行，函数没有人为指定的执行上下文。在有些情况下，**作为函数调用**的形式具有迷惑性，不仅仅是简单的函数名后面加括号来执行。<br/>
#### 1、明确的作为函数调用
&emsp;&emsp;明确的作为函数调用是指形如func(para)形式的函数调用。**作为函数调用**的情况下this在严格模式下为undefined，在非严格模式下指向全局对象（在浏览器环境下为Window对象）如下代码所示：<br/>
```js
var a = 1;
function test1 () {
    var a = 2
    return this.a
}
test1() // 1
```
```js
'use strict'
var a = 1;
function test1 () {
    var a = 2
    return this.a
}
test1() // Uncaught TypeError
```
&emsp;&emsp;以函数调用形式的函数通常不使用this，但是可以根据this来判断当前是否是严格模式。如下代码所示，在严格模式下，this为undefined，strict为true；在非严格模式下，this为全局对象，strict为false。<br/>
```js
var strict = (function () {
    return !this
})()
```
#### 2、对象作为桥梁找到方法
&emsp;&emsp;通过对象调用的函数称为方法，但是通过对象找到方法并不执行属于**作为函数调用**的情况。如下代码所示：<br/>
```js
var a = 1;

function test() {
    console.log( this.a );
}

var obj = {
    a: 2,
    test: test
};

var func = obj.test;

func(); // 1
```
&emsp;&emsp;上述代码中，obj.test是通过obj对象找到函数test，并未执行，找到函数之后将变量func指向该函数。obj对象在这个过程中只是起到一个找到test地址的桥梁作用，并不固定为函数test的执行上下文。因此var func = obj.test;执行的结果仅仅是变量func和变量test指向共同的函数体而已，因此func()仍然是**作为函数调用**，和直接调用test一样。<br/>
&emsp;&emsp;当传递回调函数时，本质也是**作为函数调用**。如下代码所示：<br/>
```js
var a = 1

function func() {
    console.log( this.a );
}

function test(fn) {
    fn();
}

var obj = {
    a: 2,
    func: func
};

test( obj.func ); // 1
```
&emsp;&emsp;函数参数是以值传递的形式进行的，obj.func作为参数传递进test函数时会被复制，复制的仅仅是指向函数func的地址，obj在这个过程中起到找到函数func的桥梁作用，因此test函数执行时，里面的fn是作为函数调用的。<br/>
&emsp;&emsp;接收回调的函数是自己写的还是语言内建的没有什么区别，比如：<br/>
```js
var a = 1;

function test() {
    console.log( this.a );
}

var obj = {
    a: 2,
    test: test
};

setTimeout( obj.test, 1000 ); // 1
```
&emsp;&emsp;setTimeout的第一个参数是通过obj对象找到的函数test，本质上obj依然是起到找到test函数的桥梁作用，因此test依然是作为函数调用的。<br/>
#### 3、间接调用传递null或undefined作为执行上下文
&emsp;&emsp;函数的间接调用是指通过call、apply或bind函数明确指定函数的执行上下文，当我们指定null或者undefined作为间接调用的上下文时，函数实际是**作为函数调用**的，即在严格模式下this为undefined，非严格模式下this为全局对象。<br/>
```js
var a = 1;

function test() {
    console.log( this.a );
}

test.call( null ); // 1
```
&emsp;&emsp;间接调用的目的是为了指定函数的执行上下文，那么为什么要传null或undefined使其作为函数调用呢？这是因为我们会用到这些方法的其他性质：函数call中一般不传入空值（null或undefined）；函数apply传入空值可以起到将数组散开作为函数参数的效果；函数bind可以用来进行函数柯里化。在ES6中，新增了扩展运算符‘...’，将一个数组转为用逗号分隔的参数序列，可以替代往apply函数传空值的情况。但是ES6中没有增加函数柯里化的方法，因此往函数bind中传空值的情况将继续使用。<br/>
&emsp;&emsp;在使用apply或bind传入空值的情况，一般是不关心this值。但是如果函数中使用了this，在非严格模式下能够访问到全局变量，有时会违背代码编写的本意。因此，使用一个真正空的值传入其中能够避免这类情况，如下代码所示：<br/>
```js
var empty = Object.create( null );
      
function foo(a,b) {
    console.log( "a:" + a + ", b:" + b );
}
      
foo.apply( empty, [1, 2] ); // a:1, b:2
```
### 二、作为方法调用
&emsp;&emsp;当函数挂载到一个对象上，作为对象的属性，则称该函数为对象的方法。如果通过对象来调用函数时，该对象就是本次调用的上下文，被调用函数的this也就是该对象。如下代码所示：<br/>
```js
var obj = {
    a: 1,
    test: test
};

function test() {
    console.log( this.a );
}

obj.test(); // 1
```
&emsp;&emsp;在JavaScript中，对象可以拥有对象属性，对象属性有又可以拥有对象或者方法。函数作为方法调用时，this指向直接调用该方法的对象，其他对象仅仅是为了找到this指向的这个对象而已。如下代码所示：<br/>
```js
function test() {
    console.log( this.a );
}

var obj2 = {
    a: 2,
    test: test
};

var obj1 = {
    a: 1,
    obj2: obj2
};

obj1.obj2.test(); // 2
```
&emsp;&emsp;当方法的返回值时一个对象时，这个对象还可以再调用它的方法。当方法不需要返回值时，最好直接返回this，如果一个对象中的所有方法都返回this，就可以采用链式调用对象中的方法。如下代码所示：<br/>
```js
function add () {
    this.a++;
    return this;
}

function minus () {
    this.a--;
    return this;
}

function print() {
    console.log( this.a );
    return this;
}

var obj = {
    a: 1,
    print: print,
    minus: minus,
    add: add
};

obj.add().minus().add().print(); // 2
```
### 三、作为构造函数调用
&emsp;&emsp;在JavaScript中，构造函数没有任何特殊的地方，任何函数只要是被new关键字调用该函数就是构造函数，任何不被new关键字调用的都不是构造函数。<br/>
&emsp;&emsp;当使用new关键字来调用函数时，会经历以下四步：<br/>
> 1、创建一个新的空对象  
> 2、这个空对象继承构造函数的prototype属性  
> 3、构造函数将新创建的对象作为执行上下文来进行初始化  
> 4、如果构造函数有返回值并且是对象，则返回构造函数的返回值，否则返回新创建的对象
>
&emsp;&emsp;约定俗成的是：在编写构造函数时函数名首字母大写，且构造函数不写返回值。因此一般来说，new关键字调用构造函数创建的新对象作为构造函数的this。如下代码所示：<br/>
```js
function foo() {
    this.a = 1;
}

var bar = new foo();
console.log( bar.a ); // 1
```
### 四、间接调用
&emsp;&emsp;在JavaScript中，对象中的方法属性仅仅存储的是一个函数的地址，函数与对象的耦合度没有想象中的高。通过对象来调用函数，函数的执行上下文（this指向）就是该对象。如果通过对象来找到函数的地址，就能指定函数的执行上下文，可以使用call()、apply()和bind()方法来实现。换而言之，任何函数可以作为任何对象的方法来调用，哪怕函数并不是那个对象的方法。<br/>
#### 1、call()和apply()
&emsp;&emsp;
#### 1、bind()
&emsp;&emsp;
### 五、规则的优先级
### 六、词法this
### 七、总结