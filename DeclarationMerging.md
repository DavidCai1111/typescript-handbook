# 声明合并

许多`TypeScript`中的独特的概念，都来源于描述`JavaScript`对象在类型层面所发生的事情。一个典型的例子就是`TypeScript`中“声明合并”的概念。理解它将会使你在`TypeScript`中处理遗留的`JavaScript`代码时游刃有余。它也会引出更多高级的抽象概念。

在我们解释如何进行声明合并时，我们先描述一下什么是“声明合并”。

在本文中，声明合并指的是编译器将会将多个同名的声明，合并入一个单独的定义中。一个被合并的定义拥有所有源声明的特性。并且这种合并不仅限于两个源声明，任意的数量都是可以的。

## 基本概念

在`TypeScript`中，一个声明必然属于以下三类之一：命名空间/模块，类型或者值。获取了 命名空间/模块 后，可以使用`.`来获取一个值。当创建一个类型时，这个声明不但创建了一个描述类型，还将其绑定给了一个名字。最后，如果创建了一个值的声明，创建的就是那些在`JavaScript`中也可见的值。

| 声明类型  |  命名空间 | 类型 | 值|
|----------|-----|--------|--------|
| 模块 | X |  | X|
| 类 | X | X | |
| 接口 |  |  | X |
| 函数 |  |  | X|
| 变量 |  |  | X|

理解每一个声明具体创建了什么后，将会帮助你理解当声明合并时，到底合并了些什么。

## 合并接口

最简单，也是可能最常用合并就是接口合并了。这种合并简单地将接口中声明的成员，以原有的名字组合在了一起：

```ts
interface Box {
    height: number;
    width: number;
}

interface Box {
    scale: number;
}

var box: Box = {height: 5, width: 6, scale: 10};
```

非函数成员的名字必须是唯一的。如果产生了名字冲突，编译器将会报错。

对于函数成员，声明合并将会对它们进行重载：

```ts
interface Document {
    createElement(tagName: any): Element;
}
interface Document {
    createElement(tagName: string): HTMLElement;
}
interface Document {
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
    createElement(tagName: "canvas"): HTMLCanvasElement;
}
```

这些接口将会被合并为一个，并且每个接口的成员都会维持原来的定义顺序，只是在重载时，顺序最后的接口中的函数会在重载列表的最前面：

```ts
interface Document {
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
    createElement(tagName: "canvas"): HTMLCanvasElement;
    createElement(tagName: string): HTMLElement;
    createElement(tagName: any): Element;
}
```

## 合并模块

与接口类似，同样名字的模块中的成员也会被合并。由于模块同时创建了命名空间和值，我们需要理解一下它们是怎么合并的。

在合并命名空间时，每个模块内的导出对象中的所有有关的类型定义都会先自身进行合并。产生一个内部包含了合并后接口定义的命名空间。

在合并值时，如果两个存在模块的名字相同，那么第二个模块中的导出值将会加到第一个模块上。

```ts
module Animals {
    export class Zebra { }
}

module Animals {
    export interface Legged { numberOfLegs: number; }
    export class Dog { }
}
```

等同于：

```ts
module Animals {
    export interface Legged { numberOfLegs: number; }

    export class Zebra { }
    export class Dog { }
}
```

为了更深入的理解，我们还需要明白非导出成员们发生了什么。非导出成员仅在各自的源模块中可见。这意味着，在合并后，合并后的成员们将不能看到其他模块中非导出成员。

例子：

```ts
module Animal {
    var haveMuscles = true;

    export function animalsHaveMuscles() {
        return haveMuscles;
    }
}

module Animal {
    export function doAnimalsHaveMuscles() {
        return haveMuscles;  // <-- error, haveMuscles is not visible here
    }
}
```

由于`haveMuscles`并没有被导出，所以只有`animalsHaveMuscles`函数可以在自己的模块中使用它。`doAnimalsHaveMuscles`函数，即使它会被合并，它也不能看到另一个模块中的未导出变量。

## 将模块与类，函数和枚举值合并

模块是十分灵活的，所以它可以合其他类型的声明合并。如果这么做，那么模块声明必须要放在它们后面。合并后的结果将同时拥有这两个声明类型的属性。

第一个例子是，将模块和类进行合并，这给了我们一种描述内部类的方式：

```ts
class Album {
    label: Album.AlbumLabel;
}
module Album {
    export class AlbumLabel { }
}
```

合并成员的可见性和上一节中的一致，所以我们必须导出`AlbumLabel`类，来让被合并的类能看到它。最终结果是一个具有内部类的新类。你也可以使用模块来为现存的类增加一些静态成员。

除了内部类，你可以还听说过通过创建一个函数，然后扩展函数的属性来扩展此函数的做法：

```ts
function buildLabel(name: string): string {
    return buildLabel.prefix + name + buildLabel.suffix;
}

module buildLabel {
    export var suffix = "";
    export var prefix = "Hello, ";
}

alert(buildLabel("Sam Smith"));
```

同样的，模块也可以用于拓展枚举值的静态成员：

```ts
enum Color {
    red = 1,
    green = 2,
    blue = 4
}

module Color {
    export function mixColor(colorName: string) {
        if (colorName == "yellow") {
            return Color.red + Color.green;
        }
        else if (colorName == "white") {
            return Color.red + Color.green + Color.blue;
        }
        else if (colorName == "magenta") {
            return Color.red + Color.blue;
        }
        else if (colorName == "cyan") {
            return Color.green + Color.blue;
        }
    }
}
```

## 不被允许的合并

并不是所有的合并都是被允许的。目前，类不能和其他类合并，变量不能和类合并，接口也不可以和类合并。更多关于声明合并的详情，请参阅 [这里](https://typescript.codeplex.com/wikipage?title=Mixins%20in%20TypeScript&referringTitle=Declaration%20Merging)。
