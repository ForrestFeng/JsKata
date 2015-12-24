#javascript核心概念#
javascript是一个结构非常简单的语言。从功能上讲可以认为有两种数据类型*object*和*function*，对于任意的对象使用typeof查询类型，结果要么是“object” 要么是“function”。但是从根本上讲JS只有一种数据类型*object*因为function也是一个object。

```shell javascript
var o1 = {};
var f1 = function(){}
typeof o1 // "object"
typeof f1 // "function"
```

##object
类型是object数据其内容由key:value定义,就如同一个字典。

比如：
```javascript
var o1 = {}
var o2 = {
	data1: "A",
	data2: "B"
}
```

就定义了两个类型是object的数据。因为o1和o2类型为object，在习惯上我们也叫o1和o2为两个object。
在shell（windows中为cmd）打开node，利用console.dir和typeof研究一个JavaScript的object。

```shell
> dir = console.dir
[Function: bound ]
>
> var o1 = {}
undefined
>
> typeof o1
'object'
>
> dir(o1)
{}
undefined
>
> o1.
o1.__defineGetter__      o1.__defineSetter__      o1.__lookupGetter__      o1.__lookupSetter__
o1.__proto__             o1.constructor           o1.hasOwnProperty        o1.isPrototypeOf
o1.propertyIsEnumerable  o1.toLocaleString        o1.toString              o1.valueOf
>
> o1.__proto__
{}
>
> typeof o1.__proto__
'object'
>
> dir(o1.__proto__)
{}
undefined
```
从实验可以看出o1
* 是类型为object的数据
* 每个空的object都具有12个缺省属性。
o1.__defineSetter__ 等以双下划线修饰的属性为内部使用属性。原则上JavaScript程序里不应使用这些内部属性，因为这是给JS的虚拟机V8使用的。但是作为研究我们依然可以使用它们一窥其内部结构。
o1.__proto__ 是我们要重点关注的一个内部属性，其重要性后面有叙述。实验可知道o1.__proto__和o1一样也是一个空的object, 当然它也有和o1一样也有__proto__属性了。

*但是*

```shell
> o1.__proto__.
o1.__proto__.__defineGetter__      o1.__proto__.__defineSetter__      o1.__proto__.__lookupGetter__
o1.__proto__.__lookupSetter__      o1.__proto__.__proto__             o1.__proto__.constructor
o1.__proto__.hasOwnProperty        o1.__proto__.isPrototypeOf         o1.__proto__.propertyIsEnumerable
o1.__proto__.toLocaleString        o1.__proto__.toString              o1.__proto__.valueOf

> o1.__proto__.__proto__
null
> dir(o1.__proto__.__proto__)
null
undefined
>



f.__proto__.constructor.__proto__  === f.__proto__
true
f.__proto__.__proto__.constructor.__proto__ === f.__proto__
true




## JavaScrit *new* 关键字是怎么工作的

### 构造器（constructor）
构造器就是一个函数，只是使用的时候在之前加了一个new关键字。JS语言并没有区分构造器和函数。一个函数即可以作为构造器使用也可以像普通的函数一样调用，你可以随便怎么使用它。

作为构造器使用函数前要加关键字new。
```javascript
var Person = function Person(name){
	this.name = name;
}

var p1 = new Person('Jack');
```

构造器语义下究竟发生了什么呢？

var p1 = new Person('Jack') 等价于一下几行代码

```
* var tmpobject = {};
* Person.prototype.constructor.call(tmpobject, 'Jack'); 
* tmpobject.__proto__ = Person.prototype;
* p1 = tmpobject;
```
p1 构造完成就是一个 Person的实例了。
```
p1 instanceof Person //true
```

以下分步解释

### var p1 = P{}; 首先创建一个新的空的对象。不多解释

### Person.prototype.constructor.call(p1, 'Jack')
此谓调用Person原型的constructor方法初始化p1。由于Person.prototype.constructor指向Person方法本身。也可以这样写
Person.call(p1, 'Jack')
就是以p1作为上下文调用Person的方法，因此Person方法里的this指的就是p1.
p1.name = name; 就是给p1属性name赋值，其值由name参数传入。


### p1.__proto__ = Person.prototype 
此谓设置p1的__protp__属性指向Person的prototype。这一步很重要因为p1.__proto__属性使得p1能够继承Person.prototype的属性。

当以上三步执行完毕p1就是一个Person的实例了

```
p1.constructor == Person //true
```
但是这个constructor属性不同于p1的显式定义的属性。当你枚举（enumerate）p1的属性的时候constructor并不包含其中。尝试显示的设置p1的constructor属性

```
var Cat = function Cat(){ 
	console.log("I am a cat");
}

p1.constructor = Cat;

p1.constructor == Cat; 	// true
p1 instanceof Cat; 		// false
p1 instanceof Person; 	// true

```
我们可以显式的改p1的constructor属性但却改变不了p1是Person的实例的事实。那p1的属性constructor是怎么来的呢?

p1的constructor属性来自于 p1.__proto__. 
__proto__是任何一个JS对象都有的属性。当p1的某个属性被访问的时候，p1先枚举自己的属性如果找到了就返回该属性，如果找不到p1就委托p1.__proto__指向的object去找这个属性。__proto__用同样的逻辑找该属性，找到了返回，找不到继续委托自己的__proto__对象。此逻辑一直重复直至一个根对象，该对象__proto__属性值为null，这时就会返回undefined。
这种基于__proto__解决属性访问的范式就是原型继承。所以JS是一种基于原型继承的语言。

那p1的__proto__属性指向谁呢？p1.__proto__指向 Person的prototype属性。（注意__proto__和prototype属性是名字相似，却是两个不同的属性）

__proto__属性任何JS的对象都具有，和语言的实现相关，不在编程中直接使用，虽然你可以使用。
prototype属性，每一个function都会自动获得一个prototype的属性。函数就是一个特殊的object，就像通常的ojbect一样（{}）函数也可以有自己的属性, prototype是函数对象自动获得的属性，当然也可以给函数显式的添加更多属性。

```
var F = function(){}
F.createDate = '2015.10.19'. 	// F.createDate
F.author = 'Forrest'			// F.author
```

函数Person的prototype就是一个普通的JS对象，该对象有一个constructor的属性指向函数Person本身。
```
typeof Person.prototype 			// "object"
p1.__proto__ == Person.prototype  	// true
Person.prototype.constructor == Person //true
p1.__proto__.constructor == Person //true
```

因为p1的原型指向Person.prototype，从表面看就是当p1构建成功之后就自动“继承”了所有Person.prototype的属性。这里“继承”并不是简单的复制而是委托。p1并不复制Person.prototype的属性到p1，而是在运行时把p1不显式拥有的属性访问委托给Person.prototype。

也因此，只要改变Person的prototype就可以在运行时动态的改变p1行为。比如何以往Person.prototype里添加一个属性。

```
Person.age = 18;
Person.prototype.sayHello = function(){
	console.log('Hello ' + this.name);
}

p1.age; 		// 18
```
只要我们想要，我们随时可以override p1 的age属性。
```
p1.age = 16;
p1.age; 		// 16
(new Person('Jerry')).age; // 18
```

同样的道理我们也可给p1动态的添加一个方法。说到底一个方法无非就是一个函数赋值到了一个对象的属性（property）上。
```
Person.prototype.sayHello = function(){
	console.log('Hello ' + this.name);
}

p1.sayHello(); 					// Hello Jack
(new Person('Tom')).sayHello(); // Hello Tom
```

注意：
如上所述，使用new关键字可以可以得到由Person函数构建的对象。如果constructor函数(Person）最后返回其他对象，则Person函数构建的tmpobject对象会被丢弃掉，p1指向函数返回的对象。但是这通常不是我们想要的，废了半天劲构建tmpobject最后丢掉这不是找抽么。所以你如果定义一个函数要作为构造器请不要返回任何东西。

## 总结
JS的prototye链条和其他的语言工作原理有所不同。说到底它很简单，当你理解了它之后，你就可以做很多看起来非常疯狂的事情。


### JS实现经典的继承语法
如果我们想要实现一个对象继承另一对象的属性（但是可以override这些对象）该怎么实现呢？
试着实现Bird和Eagle两个constructor。要求实现fly方法，Eagle要override Bird的方法, 且如下代码能够工作。

```
var bird = new Bird();
bird.fly();  	// Bird flying

var eagle = new Eagle();
eagle.fly(); 	// Eagle flying

Bird.eat = function(){ console.log("Bird eating");}
eagle.eat();  	// Eagle eating
```

```
var Bird = function Bird(name){
    this.name = name;
	this.fly = function(){
		console.log("Bird flying");
	}
}


var Eagle = function Eagle(name){
    this.name = name;
	// copy Bird property to Eagle
	for(var property in this.prototype){
		console.log('Copy property:'+property);
		Eagle.property = Bird.property;

	}
}


### JS给已有类型加上新方法
var s1 = new String('ABC');
如何给s1加上一个方法sayHello()?
```



```
JavaScript Prototype Inheritance Diagram (edit on http://asciiflow.com/)

P  = prototype
_p = _proto_
C  = constructor

 var Parent = function(name) {
   this.name = name;
   Parent.prototype.sayHello = function(){
     console.log('Hello ' + this.name);
   }
 }
 JavaScript Prototype Inheritance Diagram


 var Parent = function(name) {
   this.name = name;
   Parent.prototype.sayHello = function(){
     console.log('Hello ' + this.name);
   }
 }
 var parentInstance = new Parent('The Parent')

  +-----------------+           +----------+                 +-----------+               +----------+
  | parentInstance{}|           | Parent() |                 | Function()|               | Object() |
  +-------+---------+           +-+---^--+-+                 +--+-+--^---+               +-+--+--+--+
          |                       |   |  |                      | |  |                     |  |  |
          _p                      P   C  _p                     P _p C                    _p  P  C
          |                       |   |  |                      | |  |                     |  |  |
          |                       |   |  |                      | |  |                     |  |  |
          |                       |   |  |                   +--v-v--+----+                |  |  |
          |                       |   |  +-------------------> function() <----------------+  |  |
          |                       |   |                      +-----+------+                   |  |
          |               +-------v---+---------+                  |                          |  |
          +---------------> Parent.prototype {} |                  _p                         |  |
                          +----------+----------+                  |                          |  |
                                     |                       +-----v------+                   |  |
                                     _p                      |  Object {} <-------------------+  |
                                     +----------------------->            <----------------------+
                                                             +--^--+------+
                           +-----------------+                  |  |
                           |  emptyObject {} +------------------+  _p
                           +-----------------+    _p               |
                                                              +----v----+
                                                              |  null   |
                                                              +---------+

```





