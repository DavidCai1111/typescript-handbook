# 泛型

软件开发的一个主要部分就是不仅仅只构建具有一成不变API的组件，更是具有可复用性的组件。不仅仅兼容当前数据类型，还兼容日后数据类型的组件，将使构建大型系统更为轻松。

在`C#`和`Java`中，构建可复用组件的主要武器就是“泛型”，它让一个组件可以兼容许多类型的对象，使得用户可以使用自己的类型来运行组件。

## 泛型的Hello World

我们以一个“Hello World”泛型开始。这个函数会返回传入的参数。你可以把它当成`echo`命令的复刻。

如果没有泛型，那么我们只能明确地指定参数的类型：

```ts
function identity(arg: number): number {
    return arg;
}
```

或者，我们可以使用`any`类型：

```ts
function identity(arg: any): any {
    return arg;
}
```

使用`any`的确可以达到泛型的效果，但是这样做的话，我们就丢失了返回值的类型。如果我们传入一个数字，但返回值的依然是`any`类型。

为了能获取到返回值的类型，我们需要有一种方式来捕获参数的类型。这里，我们使用一个类型变量，这是一种特殊的不表示值而是表示类型的变量。

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

现在，我们添加了一个类型变量`T`。这使得我们可以捕获到用户传递的参数类型（如`number`）。例子中，我们使之作为了返回值类型。这样，我们就做到了返回值类型和用户传入的参数类型一致。

我们将这个版本的`identity`函数称作一个泛型函数。与使用`any`类型不同，这个函数与第一个指定参数和返回值类型为`number`的版本一样精确。

一旦我们定义了泛型函数`identity`，我们有两种办法调用它。第一种就是传入所有参数，包括类型参数：

```ts
var output = identity<string>("myString");  // type of output will be 'string'
```

上述例子中，我们在`<>`中传入`string`，来明确地将`T`设置为字符串。

第二种方式更普遍一些。我们使用“类型参数推导”，即我们让编译器基于我们传入的参数类型，自动地为我们设置变量`T`：

```ts
var output = identity("myString");  // type of output will be 'string'
```

注意，我们并没有明确将类型传入`<>`中，编译器会查找`"myString"`值的类型，然后将它赋值给T。这有助于减少代码量。但当例子更为复杂时，“类型参数推导”可能会失败，届时你需要使用第一种方式，明确地指定`T`的值。

## 泛型类型变量

当你开始使用泛型时，你会注意到当你创建如`Identity`这样的函数后，编译器为了保证函数的正确性，它只会允许你进行一些所有类型都共有的操作。因为，它可能是任何的类型。

让我们继续以`identity`函数为例子：

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

如果我们想在控制台打印出`arg`的`length`属性，我们可能会这么写：

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

当我们这么做后，编译器会给我们一个报错，因为并不能确保`arg`一定是一个包含多个成员的类型。你可能传递一个`number`类型的参数，而它并没有`length`属性。

如果要明确地表示参数是一个数组，那么我们应该直接将参数描述为`T`的数组。这样，它就一定会有`length`属性：

```ts
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

我们也可以这么写：

```ts
function loggingIdentity<T>(arg: Array<T>): Array<T> {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

如果你使用过其他一些编程语言，那么你可能已经对这样的语法十分熟悉了。在下一章中，我们将展示如果创建如`Array<T>`这样的泛型类型。

## 泛型类型

在上一节里，我们创建了泛型函数`identity`，它可以搭配许多不同类型的参数。在这一节里，我们将会探索函数类型本身，以及如何创建泛型接口。

泛型函数类型与普通函数类型相似，只是将参数类型置于最前面：

```ts
function identity<T>(arg: T): T {
    return arg;
}

var myIdentity: <T>(arg: T)=>T = identity;
```

我们也可以使用不同泛型类型参数名：

```ts
function identity<T>(arg: T): T {
    return arg;
}

var myIdentity: <U>(arg: U)=>U = identity;
```

我们还可以将泛型类型作为一个对象类型书写：

```ts
function identity<T>(arg: T): T {
    return arg;
}

var myIdentity: {<T>(arg: T): T} = identity;
```

下面我来看看泛型接口，我们将上述例子中的类型写成一个泛型接口：

```ts
interface GenericIdentityFn {
    <T>(arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

var myIdentity: GenericIdentityFn = identity;
```

我们可能希望将函数的类型参数变成整个接口的类型参数。这使得该类型参数对接口的所有成员可见。

```ts
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

var myIdentity: GenericIdentityFn<number> = identity;
```

注意上述例子中的变化。我们不再描述一个泛型函数，而是将之作为了泛型类型的一部分。当我们使用`GenericIdentityFn`时，我们需要明确地指明参数类型（例子中是`number`）。理解何时将类型参数放在函数，以及何时放在接口上，将会有助于你理解类型的哪一部分是需要进行泛型的。

除了泛型接口，我们还可以创建泛型类。但是，不能创建泛型枚举值和泛型模块。

## 泛型类

泛型类与泛型接口类似，泛型类需要在它的名字后的`<>`中，添加类型参数：

```ts
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

var myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

这是一个非常宽泛的`GenericNumber`类，你可能留意到，除了`number`类型外，我们还可以传递其他的如字符串或更复杂的对象：

```ts
var stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) { return x + y; };

alert(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

与接口一样，将类型参数置于类级别，可以保证所有类成员得到的都是相同的类型。

如我们在讨论类时提到的，一个类有静态部分和实例部分。泛型类只会影响它的实例部分。所以在使用时，静态部分将不能使用泛型类的类型参数。

## 限制泛型

回想一下之前的一个例子，我们想要取得`arg`参数的`.length`属性，但是编译器并不能保证它有一个`.length`属性，所以它警告我们不能做这样的假设：

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

我们想要将此泛型函数限制为只能传入具有`.length`属性的参数。只要参数有这个属性，那么我就允许它被传入。为了能这么做，我们需要限制`T`。

我们将创建一个接口来描述我们的限制，这里，我们创建一个只有`.length`属性的接口，然后我们使用`etends`关键字，让`T`继承这个接口，来实现我们的限制：

```ts
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

这样以后，我们的泛型函数就是受限制的了，它将不再允许传入没有`.length`属性的参数：

```ts
loggingIdentity(3);  // Error, number doesn't have a .length property

loggingIdentity({length: 10, value: 3});  
```

### 在泛型限制中使用类型参数

在一些情况下，声明一个限制另一个类型参数的类型参数是有很有用的，例子：

```ts
function find<T, U extends Findable<T>>(n: T, s: U) {   // errors because type parameter used in constraint
  // ...
}
find (giraffe, myAnimals);
```

你也可以直接替换掉泛型限制：

```ts
function find<T>(n: T, s: Findable<T>) {   
  // ...
}
find(giraffe, myAnimals);
```

注意：上面两个函数并不是完全相同的。第一个函数的返回值可以是`U`类型的，而第二个函数则做不到。

### 在泛型中使用类

在`TypeScript`中创建工厂函数时，你可能需要将类作为参数，例子：

```ts
function create<T>(c: {new(): T; }): T {
    return new c();
}
```

一个更高级的例子是使用`prototype`属性来推导类的关系：

```ts
class BeeKeeper {
    hasMask: boolean;
}

class ZooKeeper {
    nametag: string;
}

class Animal {
    numLegs: number;
}

class Bee extends Animal {
    keeper: BeeKeeper;
}

class Lion extends Animal {
    keeper: ZooKeeper;
}

function findKeeper<A extends Animal, K> (a: {new(): A;
    prototype: {keeper: K}}): K {

    return a.prototype.keeper;
}

findKeeper(Lion).nametag;  // typechecks!
```
