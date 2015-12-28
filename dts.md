# 书写 .d.ts 文件

当使用外部的`JavaScript`库时，你将需要一个描述文件（.d.ts）来描述库的API。这里讲列举一些书写定义文件的概念，然后给出一些例子。

## 指导说明

### 工作流程

最佳的书写`.d.ts`文件的方式是照着库的文档来写，而不是源码。照着文档写描述文件，保证了你所写的并没有牵涉到具体的实现细节，并且文档往往比源码更为易懂。下面的例子都将假设你正在照着文档书写描述文件。

### 命名空间

当定义接口时，你可以选择是否将这些类型放入一个模块里。如果使用者经常声明这种类型的变量或参数，并且这个类型没有和其他类型有命名冲突，那么可以将其放入全局命名空间里。但是，如果这个类型常常并不是被直接引用的，而且不能够很好地被唯一命名，那么将其放入模块中更为合适。

### 回调

许多`JavaScript`库的API接受一个函数作为参数，并且在晚些时候以固定的参数执行它。当为这类API定义类型时，不要将函数参数设置为可选的。你应该站在提供者的角度，而不是使用者的角度。

### 扩展性和声明合并

当书写描述文件时，至关重要的一点是铭记`TypeScript`扩展现存对象的规则。你可以使用一个匿名类型或一个接口类型来描述一个变量：

#### 以匿名类型描述的变量：

```ts
declare var MyPoint: { x: number; y: number; };
```

#### 以接口类型描述的变量：

```ts
interface SomePoint { x: number; y: number; }
declare var MyPoint: SomePoint;
```

对于使用方来说，它们两个的效果是一样的。但是`SomePoint`接口，可以在之后以合并的方式被扩展：

```ts
interface SomePoint { z: number; }
MyPoint.z = 4; // OK
```

你是否扩展它完全取决于你的需求。目的仅仅是为了明确地表达库的意图。

### 类的分解

`TypeScript`中的类创建了两种不同的类型：第一种是实例类型，它定义了类的实例拥有的成员，还有一种是构造函数类型，它定义了类构造函数拥有的成员。构造函数类型又被成为“静态部分”类型。

你可以通过`typeof`关键字来引用一个类的静态部分，这在一些时候是非常有用的。

以下两个例子的效果，在使用者的角度几乎都是一样的：

#### 标准写法

```ts
class A {
    static st: string;
    inst: number;
    constructor(m: any) {}
}
```

#### 分解写法

```ts
interface A_Static {
    new(m: any): A_Instance;
    st: string;
}
interface A_Instance {
    inst: number;
}
declare var A: A_Static;
```

区别如下：

 - 标准类可以使用`extends`继承。而分解类不可以。这在以后的`TypeScript`版本里可能会有所改变。
 - 标准和分解类在之后都可以通过声明合并来进行拓展。
 - 分解类可以添加实例成员，标准类不可。
 - 使用分解类时，你需要为它多起一些合理的类型名。

### 命名准则

大体上，不需要在接口前添加前缀`I`（如`IColor`）。因为在`TypeScript`中，接口的概念比在`C#`和`Java`中都宽泛，所以`I`前缀往往不是那么有用。

## 例子

让我们来看些例子。每一个例子，都会提供一个简单使用示例，然后是它的描述代码。如果有多个好的表达方法，那么它们都会被列出。


### 可选对象

#### 示例

```ts
animalFactory.create("dog");
animalFactory.create("giraffe", { name: "ronald" });
animalFactory.create("panda", { name: "bob", height: 400 });
// Invalid: name must be provided if options is given
animalFactory.create("cat", { height: 32 });
```

#### 描述

```ts
module animalFactory {
    interface AnimalOptions {
        name: string;
        height?: number;
        weight?: number;
    }
    function create(name: string, animalOptions?: AnimalOptions): Animal;
}
```

### 带属性的函数

#### 示例

```ts
zooKeeper.workSchedule = "morning";
zooKeeper(giraffeCage);
```

#### 描述

```ts
// Note: Function must precede module
function zooKeeper(cage: AnimalCage);
module zooKeeper {
    var workSchedule: string;
}
```

### 可被`new`，又可被直接调用的函数

#### 示例

```ts
var w = widget(32, 16);
var y = new widget("sprocket");
// w and y are both widgets
w.sprock();
y.sprock();
```

#### 描述

```ts
interface Widget {
    sprock(): void;
}

interface WidgetFactory {
    new(name: string): Widget;
    (width: number, height: number): Widget;
}

declare var widget: WidgetFactory;
```

### 全局或外部未知的库

#### 示例

```ts
// Either
import x = require('zoo');
x.open();
// or
zoo.open();
```

#### 描述

```ts
module zoo {
  function open(): void;
}

declare module "zoo" {
    export = zoo;
}
```

### 外部模块里的单个复杂对象

#### 示例

```ts
// Super-chainable library for eagles
import eagle = require('./eagle');
// Call directly
eagle('bald').fly();
// Invoke with new
var eddie = new eagle(1000);
// Set properties
eagle.favorite = 'golden';
```

#### 描述

```ts
// Note: can use any name here, but has to be the same throughout this file
declare function eagle(name: string): eagle;
declare module eagle {
    var favorite: string;
    function fly(): void;
}
interface eagle {
    new(awesomeness: number): eagle;
}

export = eagle;
```

### 回调

#### 示例

```ts
addLater(3, 4, (x) => console.log('x = ' + x));
```

#### 描述

```ts
// Note: 'void' return type is preferred here
function addLater(x: number, y: number, (sum: number) => void): void;
```
