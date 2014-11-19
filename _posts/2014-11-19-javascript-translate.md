##(翻译)真假性、相等性 和 JavaScript
看到一篇很好的[文章](http://javascriptweblog.wordpress.com/2011/02/07/truth-equality-and-javascript/#more-2108)，于是自己动手翻译过来＝。＝
下面是正文啦。


即使你不是一个JavaScript新手，你可能也会被这个困扰：

```
if ([0]){
	console.log([0] == true); //false
	console.log(!![0]);  //true
}
```
或者这个...

```
if("potato"){
	console.log("potato" == false);  //false
	console.log("popato" == true);  //false
}
```

好消息是存在一个标准并且所有的浏览器都遵循它。一些人会告诉你避免这样写。但是我希望说服你这是可以理解的，不必逃避。

X是真的吗？X等于Y吗？关于真假性和相等性涉及到JavaScript的三个核心区域:条件语句和操作符(if,&&，||),相等操作符(==),和严格相等操作符(===)。让我们看看在以下情况会发生什么。

###条件语句
在JavaScript中，所有的条件语句和操作符都遵从以下准则。我们将使用```if```语句作为例子。

```if```语句会遵从ES5 中定义的```ToBoolean```方法，将表达式转换为```boolean```值，之后再使用这个作为结果。
以下是算法：


参数类型 |       结果
--------|-----------------------------
undefined | false
Null | false
Boolean| 结果由参数决定
Number | false(参数为+0,-0,NaN);true(其余)
String| false(长度为0的空字符串)
Object| true

这就是JavaScript用来决定一个常规值是真还是假的规则。

现在我们能够明白，在之前的例子中，```if([0])	``` 语句会允许进入随后的语句体中：数组是对象，而对象永远为真。

下面是一些其他的例子。一些结果可能会让你吃惊，但是它们全都遵从以上的简单法则。


```
var trutheyTester = function(expr) {
    return expr ? "truthey" : "falsey"; 
}

trutheyTester({}); //truthey (an object is always true)

trutheyTester(false); //falsey
trutheyTester(new Boolean(false)); //truthey (an object!)

trutheyTester(""); //falsey
trutheyTester(new String("")); //truthey (an object!)

trutheyTester(NaN); //falsey
trutheyTester(new Number(NaN)); //truthey (an object!)


```

###相等操作符(==)

```==```操作符的规则非常的宽松。即使两个值是不同的类型也有可能相等，因为```==```在比较之前，会将两个变量强制转换为一种类型(通常是Number)。许多开发者觉得这有点吓人，不止一个知名的JS开发者呼吁避免使用```==```操作符。

逃避策略让我感到厌烦因为如果你不能从内部了解一门语言那么你就不能掌控它——恐惧和逃避是知识的敌人。而且假装```==```不存在并不会代表你再也见不到它，因为它无处不在。

无论如何，让我们看一下ECMA是怎么定义```==```的吧。它真的没有那么恐怖。只需要记住```undefined```和```null```相等，并且大多数的类型都会先转换为一个```number```型再进行比较。


x类型 |    y类型|   结果
--------|---------|--------------------
x 和 y 是同一类型| | 参看严格相等(===)算法
null|Undefined| true
Undefined|null | true
Number| String| x==toNumber(y)
String|Number |toNumber(x)==y
Boolean|(any)|toNumber(x)==y
(any)|Boolean|x==toNumber(y)
String or Number|Object|x==toPermitive(y)
Object|String or Number|toPrimitive(x)==y
otherwise...| | false

如果结果仍是表达式，递归使用算法之道结果是boolean。 ```toNumber``` 和 ```toPrimitive``` 算法如下：

>>toNumber

参数类型|结果
---------|---------
Undefined|NaN
Null| +0
Boolean| true->1;   false->+0
String|"abc"->NaN;  "123"->123
Object|1.->toPrimitive 2.->toNumber

>>toPrimitive

参数类型|结果
------|------------
Object|如果 ```Object.valueOf()```返回一个原始类型，直接返回；否则，如果```Object.toString()```返回原始类型，返回；否则，报错。
其他| 返回自身

这里有一些例子。

#### [0] == true;

```
//equlity check
[0] == true;

//how it works
//使用 toNumber 转换 Boolean
[0] == 1;
//使用 toPrimitive 转换 Object
//[0].valueOf() 非原始类型..
//[0].toString() -> "0"
"0" == 1;
//使用toNumber 转换 String
0 == 1;//false
```
##### "potato" == true;

```
//equlity check
"potato" == true;

//how it works
//使用 toNumber 转换 Boolean
"potato" == 1;
//使用toNumber 转换 String
NaN == 1;//false
```
##### "potato" == false;

```
//equlity check
"potato" == false;

//how it works
//使用 toNumber 转换 Boolean
"potato" == 0;
//使用 toString 转换 String
NaN == 0;//false

```

#### "object with valueOf"
```
//equlity check
crazyNumeric ＝ new Number(1);
crazyNumeric.toString = function(){return "2";};
crazyNumeric == 1;

//how it works
//使用 toPermitive 转换 Object
// valueOf() 返回原始类型1
1 == 1;//true
```

#### "object with toString"
```
//equlity check
var crazyObj = {
	toString:function(){return "2";}
}
crazyObj == 1;

//how it works
//使用 toPermitive 转换 Object
//valueOf()返回Object，继续使用toString 转换Object
"2" == 1;
//使用toNumber 转换 String
2 == 1;//false

```

###严格相等操作符(===)
这个很简单。如果是不同类型，结果永远是false。如果是同一种类型，那么依据下面的守则：对象引用必须指向同一个对象，`String` 必须包含相同的字符，其他原始类型必须有相同的值。`NaN` ,`null` 和 `undefined` 不会等于(===)其他类型。`NaN`甚至不会等于其本身。

参数类型|参数的值|结果
------|---------|-----------
typeof(x) 与 typeof(y)不同||false
Undefined or Null || true
Number|x 与y 的值一样 |true
String|x 与 y 一样的字符|true
Boolean|x 与 y 同为true 或者false|true
Object|x 与 y 指向同一个对象|true
otherwise|  |false



###overskill

```
//unnecessary
if(typeof myVar === 'fucntion');

//better
if(typeof myVar == 'function');
```

..因为 `typeof`返回一个`string`，这个操作永远是比较两个字符串。


```
//unnecessary
var missing = (myVar === undefined || myVar === null);

//better
var missing = (myVar == null);

```

.. `null` 和 `undefined` 是 相等(==)的。
































	
