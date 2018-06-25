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
&emsp;&emsp;函数的间接调用是指通过call、apply或bind函数明确指定函数的执行上下文，当我们指定null或者undefined作为间接调用的上下文时，函数实际是**作为函数调用**的。但是有一点需要注意：call()和apply()在严格模式下传入空值则上下文为空值，并不是因为遵循**作为函数调用**在严格模式下执行上下文为全局对象的规则，而是因为在严格模式下call()和apply()的第一个实参都会变成this的值，哪怕传入的实参是原始值甚至是null或undefined。<br/>
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
> 1、创建一个新的空对象。  
> 2、这个空对象继承构造函数的prototype属性。  
> 3、构造函数将新创建的对象作为执行上下文来进行初始化。  
> 4、如果构造函数有返回值并且是对象，则返回构造函数的返回值，否则返回新创建的对象。

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
&emsp;&emsp;每个函数都call()和apply()方法，函数调用这两个方法是可以明确指定执行上下文。从绑定上下文的角度来说这两个方法是一样的，第一个参数传递的都是指定的执行上下文。所不同的在于call()方法剩余的参数将会作为函数的实参来使用，可以有多个；apply()则最多只接收两个参数，第一个是执行上下文，第二个是一个数组，数组中的每个元素都将作为函数的实参。如下代码所示：<br/>
```js
var a = 1
function test(b,c) {
    console.log(`a:${this.a},b:${b},c:${c}`)
}
var obj = {
    a:2
}
test.call(obj,3,4) // a:2,b:3,c:4

var d = 11
function test2(b,c) {
    console.log(`b:${b},c:${c},d:${this.d}`)
}
var obj2 = {
    d:12
}
test2.apply(obj2,[13,14]) // b:13,c:14,d:12
```
&emsp;&emsp;在非严格模式下，call()、apply()的第一个参数传入null或者undefined时，函数的执行上下文被替代为全局对象，如果传入的是基础类型，则为替代为相应的包装对象。在严格模式下，遵循的规则是传入的值即为执行上下文，不替换，不自动装箱。如下代码所示：<br/>
```js
var a = 1
function test1 () {
    console.log(this.a)
}
test1.call(null) // 1
test1.call(undefined) // 1
test1.apply(null) // 1
test1.apply(undefined) // 1
```
***
```js
'use strict'
function test1 () {
    console.log(this)
}
test1.call(null) // null
test1.call(undefined) // undefined
test1.call(1) // 1
test1.apply(null) // null
test1.apply(undefined) // undefined
test1.apply(1) // 1
```
&emsp;&emsp;apply()有一个较为常见的用法：将数组转化成函数的参数序列。ES6中增加了扩展运算符“...”来实现该功能。如下代码所示：<br/>
```js
var arr = [1,19,4,54,69,9]

var a = Math.max.apply(null,arr)
console.log(a) // 69

var b = Math.max(...arr)
console.log(b) // 69
```
#### 2、bind()
&emsp;&emsp;bind()函数可以接收多个参数，返回一个功能相同、执行上下文确定、参数经过初始化的函数。其中第一个参数为要绑定的执行上下文，剩余参数为返回函数的预定义值。bind()函数的作用有两点：1、为函数绑定执行上下文；2、进行函数柯里化。如下代码所示：<br/>
```js
var a = 1

function func(b,c) {
    console.log(`a:${this.a},b:${b},c:${c}`)
}

var obj = {
    a: 2
}

var test = func.bind(obj,3)

test(4) // a:2,b:3,c:4
```
&emsp;&emsp;bind()方法是ES5加入的，但是我们可以很轻易的在ES3中通过apply()模拟出来，下面代码是MDN上的bind()的polyfill。<br/>
```js
if (!Function.prototype.bind) {
    Function.prototype.bind = function(oThis) {
        if (typeof this !== "function") {
            // 可能的与 ECMAScript 5 内部的 IsCallable 函数最接近的东西，
            throw new TypeError( "Function.prototype.bind - what " +
                "is trying to be bound is not callable"
            );
        }

        var aArgs = Array.prototype.slice.call( arguments, 1 ),
            fToBind = this,
            fNOP = function(){},
            fBound = function(){
                return fToBind.apply(
                    (this instanceof fNOP &&oThis ? this : oThis),
                    aArgs.concat( Array.prototype.slice.call( arguments ) )
                );
            };

        fNOP.prototype = this.prototype;
        fBound.prototype = new fNOP();

        return fBound;
    };
}
```
### 五、规则的优先级
&emsp;&emsp;函数的调用有时不只一种，那么不同调用方式的规则的优先级就最终决定了this的指向。那就让我们来比较不同调用方式的规则优先级。如下代码所示，当函数作为方法调用的时候，this指向调用方法的对象，当作为函数调用时，this指向在非严格模式下指向全局对象，在严格模式下指向undefined。因此，方法调用的优先级高于函数调用。<br/>
```js
var a = 1

var obj = {
    a:2,
    test:test
}

function test () {
    console.log(this.a)
}

var b = obj.test

obj.test() // 2
b() // 1
```
&emsp;&emsp;如下代码所示是函数作为方法调用分别和间接调用、构造函数调用作对比。由代码可知：函数作为方法调用优先级分别小于间接调用和构造函数调用。<br/>
```js
function test(para) {
    this.a = para
}

var obj1 = {
    test: test
}

var obj2 = {}
      
obj1.test( 2 )
console.log( obj1.a ) // 2

obj1.test.call( obj2, 3 )
console.log( obj2.a ) // 3

var bar = new obj1.test( 4 )
console.log( obj1.a ) // 2
console.log( bar.a ) // 4
```
&emsp;&emsp;new关键字后面是一个函数，而call()和apply()并不是返回一个函数，而是依照传入参数来执行函数，因此形如new foo.call(obj)的代码是不被允许的。ES5中的bind()返回的是一个函数，可以与new关键字同时使用。如下代码所示，bind()返回的函数用作构造函数，将忽略传入bind()的this值，原始函数会以构造函数的形式调用，传入的参数也会原封不动的传入原始函数。<br/>
```js
function test(something) {
    this.a = something;
}

var obj = {};

var bar = test.bind( obj );
bar( 2 );
console.log( obj.a ); // 2

var baz = new bar( 3 );
console.log( obj.a ); // 2
console.log( baz.a ); // 3
```
&emsp;&emsp;总之，构造函数的优先级大于间接调用，间接调用的优先级大于方法调用，方法调用的优先级大于函数调用。<br/>
### 六、词法this
&emsp;&emsp;this关键字没有作用域限制，函数的this指向调用该函数的对象，在嵌套函数汇中，如果想访问外层函数的this值，可以将外层函数的this赋值给一个变量，用词法作用域来代替传统的this机制。如下代码所示：<br/>
```js
function foo() {
    var self = this // 词法上捕获`this`
    setTimeout( function(){
        console.log( self.a )
    }, 1000 )
}

var obj = {
    a: 2
};

foo.call( obj ) // 2
```
&emsp;&emsp;ES6新增了箭头函数，箭头函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。如下代码所示，箭头函数能够将this固化，箭头函数内部没有绑定this的机制，其内部的this就是外层代码块的this。传统的this机制让很多人与词法作用域混淆，因此有了将this赋值给变量的行为，ES6只是将这种行为加以标准化而已。<br/>
```js
var a = 21

function test() {
    setTimeout(() => {
        console.log('a:', this.a)
    }, 1000)
}

test.call({ a: 42 }) // 2
```
### 七、总结
&emsp;&emsp;JavaScript中的this机制跟词法作用域没有关系，根据函数调用的方式不同，确定this指向的规则也不相同。在确定this指向时可以遵循以下步骤：<br/>
>1、函数是否为**构造函数调用**，即函数跟在new关键字后面，如果是，this就是新构建的对象。<br/>
>2、函数是否为**间接调用**，即通过call()、apply()或者bind()调用，如果是，this就是明确指定的对象。<br/>
>3、函数是否为**作为方法调用**，即通过对象来调用函数，如果是，this就是该对象。<br/>
>4、否则，即为**作为函数的调用**，在非严格模式下，this指向全局对象，在严格模式下，this为undefined。<br/>

&emsp;&emsp;可以将外层函数的this赋值给一个变量，使得内层函数以词法作用域的规则来访问该this。ES6新增的箭头函数便是使用词法作用域来决定this绑定的。<br/>