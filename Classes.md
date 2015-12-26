# 类

传统的`JavaScript`在构造可复用组件时，使用的是基于原型的继承，这对于习惯了使用基于类的面向对象编程语言的人来说，可能会有些古怪。不过，自从ES6开始，`JavaScript`也支持书写基于类的对象了。在`TypeScript`中，我们同样允许开发者书写这种基于类的代码，并且可以编译成跨浏览器，跨平台的通用代码。

## 类

让我们以一个简单的类的例子开始：

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

var greeter = new Greeter("world");
```

如果你以前写过`C#`或`Java`，那么你一定对这种语法十分的熟悉。在这里，我们生成了一个新的`Greeter`类。这个类有三个成员，一个`greeting`属性，一个构造函数，以及一个`greet`方法。

你可能注意到了，在我们访问内部成员时，我们总是在前面加上`this.`。这表示我们将访问一个成员变量。

在最后一行，我们使用`new`创建了一个`Greeter`类的实例。这将会调用我们先前声明的类的构造函数。

## 继承

`TypeScript`使用了最普遍的面向对象模型。所以，它也支持通过继承来扩展类。

例子：

```ts
class Animal {
    name:string;
    constructor(theName: string) { this.name = theName; }
    move(meters: number = 0) {
        alert(this.name + " moved " + meters + "m.");
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(meters = 5) {
        alert("Slithering...");
        super.move(meters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(meters = 45) {
        alert("Galloping...");
        super.move(meters);
    }
}

var sam = new Snake("Sammy the Python");
var tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```

上面的例子中，我们使用了`extends`关键字，来创建了一个子类。`Horse`和`Snake`子类都是继承与`Animal`类的，它们包含`Animal`类的所有特性。

子类也可以重载父类的方法，在上面的例子中，`Snake`和`Horse`都重新定义了它们各自的`move`方法。

## 私有/公开 描述符

### 默认是公开的

你可能通过上面的例子已经察觉到，我们不必使用`public`关键字来描述公开的类的属性和方法。而在像`C#`这样的语言里，你都必须明确地为公开成员添加`public`标识。不过在`TypeScript`中，成员默认就是公开的。

你可能需要将一些成员标识为私有的，来保证它们不会被外部代码所看见。我们来修改一下之前的`Animal`类：

```ts
class Animal {
    private name:string;
    constructor(theName: string) { this.name = theName; }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}
```

### 理解私有

`TypeScript`有一个结构化的类型系统。当我们比较两个不同的类型时，不论它们来自哪里，如果它们的成员的类型是兼容的，那么我们说这些类型本身是兼容的。

但是当比较的一方是私有的成员时，我们便会区别对待它们。如果一方是私有成员，那么另一方必须是指向同一个私有成员时，才算兼容。

例子：

```ts
class Animal {
    private name:string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
  constructor() { super("Rhino"); }
}

class Employee {
    private name:string;
    constructor(theName: string) { this.name = theName; }
}

var animal = new Animal("Goat");
var rhino = new Rhino();
var employee = new Employee("Bob");

animal = rhino;
animal = employee; //error: Animal and Employee are not compatible
```

在上面的例子里，我们有`Animal`类和`Rhino`类，并且`Rhino`是`Animal`的子类。我们还有另外一个类叫作`Employee`，它与`Animal`类的外观类似。我们分别创建了它们的实例，并且将其中一个赋值给另外一个，然后在将`employee`赋值给`animal`时，我们就会得到一个报错。因为在`Animal`和`Rhino`中，它们的私有属性`name`都是来自于`Animal`类中的`private name: string`，所以它们是兼容的。但是，虽然`Employee`也有私有成员`name`，但由于它们并不是在同一处定义的，所以它们并不兼容。

### 参数属性

`public`和`private`关键字还给予了你一个快速初始化成员的方法，通过使用参数属性，你只需一步就可以创建和初始化一个成员。下面的例子是上文例子的改版。注意我们将声明和赋值都放在了`private name: string`参数上。

```ts
class Animal {
    constructor(private name: string) { }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}
```

由于使用的是`private`关键字，所以我们初始化了一个私有成员。当然，使用`public`时，效果也是类似的。

## 访问器

`TypeScript`支持`getters/setters`来作为属性的访问器。

以下是一个简单的使用`get`和`set`的例子，但是首先，让我们先从没有它们的代码开始：

```ts
class Employee {
    fullName: string;
}

var employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

以上的例子里，允许用户直接编辑`fullName`属性是有些危险的。

在下面的例子中，我们会在用户修改`fullName`之前，检查一个`passcode`。同时，我们也为`fullName`添加一个getter，来使之能完整的运作：

```ts
var passcode = "secret passcode";

class Employee {
    private _fullName: string;
    get fullName(): string {
        return this._fullName;
    }

    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            alert("Error: Unauthorized update of employee!");
        }
    }
}

var employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

为了证明我们的访问器是正常运作的，我们可以修改`passcode`变量，当`passcode`不匹配时，我们会得到一个警告弹窗。

注意：使用访问器时，你需要将编译器的输出设置为`ECMAScript 5`。

## 静态属性

目前为止，我们只讨论了类的实例成员，这些成员在类被实例化后，才可见。我们也可以创造类的静态成员，这些成员在类的级别就可见。在下面的例子中，我们使用`static`关键字，置于`origin`属性之前，使之成为`Grid`类级别的静态属性。

```ts
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        var xDist = (point.x - Grid.origin.x);
        var yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

var grid1 = new Grid(1.0);  // 1x scale
var grid2 = new Grid(5.0);  // 5x scale

alert(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
alert(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

## 高级技巧

### 构造函数

当你在`TypeScript`中创建一个类时，实际上你创建了许多东西。第一个，就是你创建了这个类的实例的类型：

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

var greeter: Greeter;
greeter = new Greeter("world");
alert(greeter.greet());
```

上面的例子里，我们使用了`Greeter`作为其实例的类型，即`var greeter: Greeter`。当然，这种情况也出现在其他许多面向对象的语言中。

同时，我们又调用构造函数创建了另一个值。构造函数在我们使用`new`关键字时会被调用。例子：

```ts
var Greeter = (function () {
    function Greeter(message) {
        this.greeting = message;
    }
    Greeter.prototype.greet = function () {
        return "Hello, " + this.greeting;
    };
    return Greeter;
})();

var greeter;
greeter = new Greeter("world");
alert(greeter.greet());
```

上面的例子中，`var Greeter`被赋值为了一个构造函数。当我们使用`new`关键字时，我们创建了这个类的一个实例。这个构造函数也包含了所有类的静态成员。

让我们稍微修改下这个例子：

```ts
class Greeter {
    static standardGreeting = "Hello, there";
    greeting: string;
    greet() {
        if (this.greeting) {
            return "Hello, " + this.greeting;
        }
        else {
            return Greeter.standardGreeting;
        }
    }
}

var greeter1: Greeter;
greeter1 = new Greeter();
alert(greeter1.greet());

var greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";
var greeter2:Greeter = new greeterMaker();
alert(greeter2.greet());
```

在上面的例子中，`greeter1`与上面例子中的实例一模一样。我们实例化了`Greeter`类，然后使用它。

然后，我们直接使用类。我们创建了一个`greeterMaker`变量。这个变量的值就是类本身，或者说就是类的构造函数。我们使用`typeof Greeter`来定义其类型，即它的构造函数的类型。这个类型还会包含类的所有静态成员。然后，我们可以在`greeterMaker`之前使用`new`关键字，来创建一个新的实例，并且使用它。

### 将类作为接口使用

正如我们在上一节中所提到的，一个类声明其实创建了两件东西：一个代表类实例的类型，以及一个构造函数。因为类能创建类型，所以你也可以在能使用接口的地方使用它。

```ts
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

var point3d: Point3d = {x: 1, y: 2, z: 3};
```
