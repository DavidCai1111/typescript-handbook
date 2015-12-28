# 类型兼容性

`TypeScript`的类型兼容性是基于结构化子类型的。它与名义类型（nominal typing）相对立。思考以下代码：

```ts
interface Named {
    name: string;
}

class Person {
    name: string;
}

var p: Named;
// OK, because of structural typing
p = new Person();
```

在大多数的名义类型编程语言中（如`C#`或`Java`），以上代码将会得到一个报错，因为`Person`类没有明确表示它实现了`Named`接口。

`TypeScript`的结构化类型系统是基于`JavaScript`的典型编码场景所设计的。由于`JavaScript`经常使用像函数表达式或对象字面量这样的匿名对象，利用结构化类型系统，会更适用于描述它们间的关系。

### 关于可靠性的提醒

`TypeScript`的类型系统允许某些不能在编译阶段就确保安全的操作。当一个类型系统允许这些时，它会被认为是不可靠的。但`TypeScript`之所以允许这些行为是经过严格的思考的，我们将会在下文解释原因。

## 开始

`TypeScript`的结构化类型系统中，最基本的原则就是，如果`y`至少有和`x`一模一样的成员，那么`x`就是兼容`y`的。例子：

```ts
interface Named {
    name: string;
}

var x: Named;
// y’s inferred type is { name: string; location: string; }
var y = { name: 'Alice', location: 'Seattle' };
x = y;
```

为了检查`y`可否能被赋值给`x`，编译器会检查`x`中的所有属性，在`y`里是否有相匹配的。所以在例子里，`y`必须包含一个类型为`string`的`name`属性。它的确有，所以这个赋值操作是可行的。

在检查函数调用时的参数时，规则也是一样的：

```ts
function greet(n: Named) {
    alert('Hello, ' + n.name);
}
greet(y); // OK
```

值得注意的是，`y`有一个额外的`location`属性，但这并不会产生一个错误。只有目标类型（例子中为`Named`）的成员，才会被纳入兼容性检查。

编译器会递归地比较两个类型下的成员以及子成员。

## 比较两个函数

两个基本类型值和对象的比较是十分直观的。那么是时候来探讨两个函数的兼容性了。让我们以一组只有参数不同的函数开始：

```ts
var x = (a: number) => 0;
var y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error
```

当将`x`赋值给`y`时，编译器首先会查看参数列表。`y`中的每一个参数都必须在`x`中有对应的类型兼容的参数。注意，参数名不同并没有关系，仅会考虑它们的类型。在例子里，`x`中的每一个参数都在`y`中有相兼容的参数，所以这个赋值是可行的。

而第二个赋值操作则会报错。因为`y`要求有第二个参数，而`x`并没有。

你或许会疑问，为什么我们在`y = x`的例子里，会允许“丢弃”第二个参数。这是因为，在`JavaScript`中，忽略后面的部分参数是很常见的做法。例如，`Array#forEach`提供了三个参数：数组单个元素，索引，和数组本身。但是实际使用中，人们经常只传递第一个参数：

```js
var items = [1, 2, 3];

// Don't force these extra arguments
items.forEach((item, index, array) => console.log(item));

// Should be OK!
items.forEach((item) => console.log(item));
```

现在让我们来看看返回值，以下是两个只有返回值不同的函数：

```ts
var x = () => ({name: 'Alice'});
var y = () => ({name: 'Alice', location: 'Seattle'});

x = y; // OK
y = x; // Error because x() lacks a location property
```

类型系统要求源函数的返回值是目标函数返回值的子集。

### 函数参数的双边变化

在比较两个函数参数的类型时，只要源参数是可以赋值给目标参数的，或反之，赋值都能成功。这被认为是不可靠的，因为一边的函数参数可能会被描述更精确的参数类型，但是执行是却被传入一个更宽泛的类型。在实践中，这类的错误时很罕见的，并且借此实现了很多`JavaScript`模式。一个简单的例子：

```ts
enum EventType { Mouse, Keyboard }

interface Event { timestamp: number; }
interface MouseEvent extends Event { x: number; y: number }
interface KeyEvent extends Event { keyCode: number }

function listenEvent(eventType: EventType, handler: (n: Event) => void) {
    /* ... */
}

// Unsound, but useful and common
listenEvent(EventType.Mouse, (e: MouseEvent) => console.log(e.x + ',' + e.y));

// Undesirable alternatives in presence of soundness
listenEvent(EventType.Mouse, (e: Event) => console.log((<MouseEvent>e).x + ',' + (<MouseEvent>e).y));
listenEvent(EventType.Mouse, <(e: Event) => void>((e: MouseEvent) => console.log(e.x + ',' + e.y)));

// Still disallowed (clear error). Type safety enforced for wholly incompatible types
listenEvent(EventType.Mouse, (e: number) => console.log(e));
```

## 可选参数和rest参数

当比较两个函数时，可选和必选参数是可以相互交换的。源函数的额外可选参数将不会导致一个报错，目标函数的不对应的可选参数也不会导致报错。

当一个函数有rest参数时，它被视为有无限个可选参数。

这也被认为是不可靠的，因为在大多数的运行时里，可选参数的空缺往往会被强制传入一个`undefined`。

下面的例子里，一个函数接受一个回调，并且使用一个（对于程序员）可预测的，但是（对于类型系统）未知数量的参数执行：

```ts
function invokeLater(args: any[], callback: (...args: any[]) => void) {
    /* ... Invoke callback with 'args' ... */
}

// Unsound - invokeLater "might" provide any number of arguments
invokeLater([1, 2], (x, y) => console.log(x + ', ' + y));

// Confusing (x and y are actually required) and undiscoverable
invokeLater([1, 2], (x?, y?) => console.log(x + ', ' + y));
```

### 有重载的函数

当一个函数具有重载时，它重载列表中的每一个函数类型都必须与目标匹配。这保证了目标函数可以在相同的情况下被执行。

## 枚举类型

枚举类型和数字类型兼容，反之也成立。不同的枚举类型的枚举值是不兼容的。例子：

```ts
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

var status = Status.Ready;
status = Color.Green;  //error
```

## 类

类的兼容性与对象字面量和接口类似，但只有一个区别：类有静态和实例部分。当比较两个类的实例时，只有实例部分会被比较。静态部分和构造函数并不会影响兼容性。

```ts
class Animal {
    feet: number;
    constructor(name: string, numFeet: number) { }
}

class Size {
    feet: number;
    constructor(numFeet: number) { }
}

var a: Animal;
var s: Size;

a = s;  //OK
s = a;  //OK
```

### 类中的私有成员

当一个类中有私有成员时，目标类必须有来自同一出处的私有成员，才会被认作是兼容的。举个例子，子类是兼容父类的，但具有相同描述的具有私有成员的两个不同类则不兼容。

## 泛型

由于`TypeScript`使用的是结构化类型系统，类型参数只影响其作为部分成员的结果类型。例子：

```ts
interface Empty<T> {
}
var x: Empty<number>;
var y: Empty<string>;

x = y;  // okay, y matches structure of x
```

上述例子中，`x`和`y`是兼容的，因为它们的机构体里没有以不同方式使用类型参数。让我们改变一下：

```ts
interface NotEmpty<T> {
    data: T;
}
var x: NotEmpty<number>;
var y: NotEmpty<string>;

x = y;  // error, x and y are not compatible
```

这样的话，它们就有了各自独特类型的属性，就像有了非泛型属性一样。

对于没有在内部使用过类型参数的泛型，兼容性检查会将类型参数视为`any`。然后再以非泛型的方式得出检测结果：

For example,

```ts
var identity = function<T>(x: T): T {
    // ...
}

var reverse = function<U>(y: U): U {
    // ...
}

identity = reverse;  // Okay because (x: any)=>any matches (y: any)=>any
```

## 高级话题

### 子类型 vs 赋值

至今为止，我们讨论了“兼容性”，这并不是一个被定义在了语言层面的概念。在`TypeScript`中，有两种兼容性：子类型和赋值。它们仅有的不同是，赋值操作通过传递`any`的规则，拓展了子类型兼容性。

不同的情况下，`TypeScript`会选择不同的兼容性。实际生产中，甚至在`implements`和`extends`语句里，类型兼容性也是由赋值兼容性所控制的。更多信息，请参阅 [这里](http://www.typescriptlang.org/Content/TypeScript%20Language%20Specification.docx)。
