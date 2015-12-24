# 接口

`TypeScript`的一个核心守则就是专注于检查值的“形状（shape）”。在`TypeScript`中，接口承担起了这个职责。你可以使用它来命名类型，它是结构化代码的好工具，并且你也可以使用它来对外部调用代码进行约束。

## 第一个接口

一个简单的接口例子：

```ts
function printLabel(labelledObj: {label: string}) {
  console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

`TypeScript`的类型检查器会去检查`printLabel`的调用。`printLabel`有一个参数，且这个参数必须是一个包含一个名为`label`的字符串属性的对象。值得注意的是，我们实际传入的对象，其实包含了额外的属性，但类型检查器仅仅只会去检查指定的属性是否正确存在。

让我们重构一下以上代码，这次我们使用接口来描述一个必须包含一个名为`label`的字符串属性的对象：

```ts
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

接口`LabelledValue`与我们在上一个例子中所做的事是一样的。值得注意的是，我们并不需要像在其他的强类型语言中一样，传入`printLabel`的参数一定非得是一个实现了这个接口的对象。在`TypeScript`中，它们只需在形态上一致就可以了。只要我们传递的参数符合接口所列出的定义，它就是合法的。

还有一个值得注意的点是，类型检查器并不会去关注接口中属性在被定义时的顺序。你可以以任意的顺序来实现它。

## 可选属性

一个接口中的所有属性并不都是必须的。在一些情况下，它们可能并不存在。比如，当用户传递一个`option`对象参数时，可能有很多配置属性都是可选的。

可选属性例子：

```ts
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

var mySquare = createSquare({color: "black"});
```

可选属性的写法和接口中其他的属性写法很像，仅仅在冒号之前添加一个`?`即可。

可选属性的优势在于，我们即可以用它来描述可能出现的属性，同时又可以检查出那些不该存在的属性。如下面的例子中，我们在定义`createSquare`的函数体时，config的一个属性名拼错了，我们将会得到一个报错：

```ts
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.collor;  // Type-checker can catch the mistyped name here
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

var mySquare = createSquare({color: "black"});  
```

## 函数类型

接口除了可以用来定义`JavaScript`对象中各式各样的属性。它也可以用来定义函数。

在定义函数类型的接口时，语法很像函数声明，但是只有参数列表和返回值类型。


```ts
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```

一旦定义了函数接口，我们就可以像普通接口去使用它。下面的例子中，我们定义了一个函数接口，然后定义了一个函数实现了它：

```ts
var mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  var result = source.search(subString);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
}
```

在做函数接口的类型检查时，参数的名字是不必完全一样的，下面的例子也是完全合法的：

```ts
var mySearch: SearchFunc;
mySearch = function(src: string, sub: string) {
  var result = src.search(sub);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
}
```

除了检查参数之外，函数的返回值也会被检查（这里是`true`和`false`）。如果函数返回数字或字符串，那么类型检查器将会提示我们，函数的返回值与`SearchFunc`接口中所描述的不一致。

## 数组类型

既然我们可以用接口来描述函数，我们也可以用接口来描述数组。数组有一个“索引(index)”类型来描述这个数组的索引的类型，紧接着的是对应位置元素的返回值。

```ts
interface StringArray {
  [index: number]: string;
}

var myArray: StringArray;
myArray = ["Bob", "Fred"];
```

`TypeScript`支持两种索引类型：字符串和数字。一个数组同时支持两种索引也是允许的，但是有一个限制，数字索引的返回值的类型，必须是字符串索引的返回值类型的子类型。

尽管索引在描述数组和“字典（dictionary）”时很有用，但是它们也限制了返回值的类型。在下面的例子中，有属性没有符合索引中指定的返回值类型，所以会得到一个报错：

```ts
interface Dictionary {
  [index: string]: string;
  length: number;    // error, the type of 'length' is not a subtype of the indexer
}
```

## 类

### 实现一个接口

接口在`C#`或`Java`中的一大用处，就是给予一个类明确的限制。在`TypeScript`中，一样如此：

```ts
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface  {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

你也可以在一个类的接口中定义一些方法，下面的例子中，我们定义了`setTime`方法：

```ts
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface  {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

接口仅仅用于定义一个类中的公有部分。用接口来定义类的私有属性/方法，都是不允许的。

### 类中的静态部分和实例部分

在使用类和接口时，你需要记住，一个类包含两部分：静态部分和实例部分。你可能会注意到，当你在一个接口中声明构造函数，并且尝试让一个类去实现这个接口时，你会得到一个错误：

```ts
interface ClockInterface {
    new (hour: number, minute: number);
}

class Clock implements ClockInterface  {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

这是因为当类实现一个接口时，只有类的实例部分才会被检查。由于构造函数是类的静态部分，它将不会被检查。

所以，你应当直接在类中处理静态部分：

```ts
interface ClockStatic {
    new (hour: number, minute: number);
}

class Clock  {
    currentTime: Date;
    constructor(h: number, m: number) { }
}

var cs: ClockStatic = Clock;
var newClock = new cs(7, 30);
```

## 继承接口

和类一样，接口也可以相互继承。所以，你不需要在接口之间拷贝那些公有的属性了。它使你可以抽出接口之间的可重用部分：

```ts
interface Shape {
    color: string;
}

interface Square extends Shape {
    sideLength: number;
}

var square = <Square>{};
square.color = "blue";
square.sideLength = 10;
```

一个接口可以继承多个接口：

```ts
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

var square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

## 混合类型

正如我们之前提到的，接口可以描述`JavaScript`世界中的许多类型。但是因为`JavaScript`天生就是动态的，你们可能会遇到一些混合类型的对象。

以下例子是一个即使函数类型又是对象类型的`JavaScript`对象：

```ts
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

var c: Counter;
c(10);
c.reset();
c.interval = 5.0;
```

当和第三方`JavaScript`库打交道时，你可能会用上上述特性，用以完整得描述这些库。
