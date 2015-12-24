# 基本类型

为了让我们的编程更加高效，我们总是需要一些基本类型的数据，如：`number`, `string`,`structure`,`boolean`等等。`TypeScript`支持所有`JavaScript`中的基本类型，并且还有它额外添加的类型，如`enumeration`。

## Boolean

真/假值总是最基本的数据类型，在`JavaScript`和`TypeScript`（以及许多其他编程语言）中，这个类型都叫作`boolean`。

```ts
var isDone: boolean = false;
```

## Number

和在`JavaScript`中一样，所有`TypeScript`中的数字类型值都是浮点数。这些浮点数的类型名叫作`number`。

```ts
var height: number = 6;
```

## String

文本数据是又一个编程语言中的基本类型。与其他的语言一样，在`TypeScript`中，它类型名叫作`string`。与`JavaScript`中一样，`TypeScript`中的`string`类型值，被一对双引号(`"`)或单引号(`'`)所包围。

```ts
var name: string = "bob";
name = 'smith';
```

## Array

和`JavaScript`一样，`TypeScript`同样也允许你使用数组值，它的类型名为`Array`。它有两种书写方式。第一种，即在元素的类型后加上`[]`来注明这是一个此类型元素的数组：

```ts
var list:number[] = [1, 2, 3];
```

第二种方式，是使用泛型数组类型，`Array<elemType>`:

```ts
var list:Array<number> = [1, 2, 3];
```

## Enum

`enum`类型是在`JavaScript`的标准类型集合的基础之上，额外添加的类型。和`C#`语言中一样，`enum`类型赋予数字值的集合一个更友好的名字。

```ts
enum Color {Red, Green, Blue};
var c: Color = Color.Green;
```

默认情况下，`enum`类型值从`0`开始，对它的成员进行枚举。你也可以通过手动地为成员赋值，来改变它。例如，你可以让初始值变为`1`：

```ts
enum Color {Red = 1, Green, Blue};
var c: Color = Color.Green;
```

甚至你也可以手动地为所有成员赋值：

```ts
enum Color {Red = 1, Green = 2, Blue = 4};
var c: Color = Color.Green;
```

`enum`类型值的一个十分方便特性就是，你可以通过这个值对应的数字，来获取这个值。例如，我们有数字值`2`，却不知道它在`Color`中是映射到哪个值，我们便可以通过此数字来查找：

```ts
enum Color {Red = 1, Green, Blue};
var colorName: string = Color[2];

alert(colorName);
```

## Any

当我们在写程序时，我们有时可能会遇到我们并不能知道其类型的值。这些值可能来自于一些动态的上下文，比如，来自用户，或来自第三方库。在这种情况下，我们希望暂时放弃类型检查，让它们可以通过编译。我们可以通过为这些值添加类型`any`来实现：

```ts
var notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```

在与现存的`JavaScript`代码交互时，`any`类型十分有用。

`any`类型，在你仅仅能明确这个值的一部分类型信息时，也十分有用。比如，你可能会遇到一个包含不同类型值的数组：

```ts
var list:any[] = [1, true, "free"];

list[1] = 100;
```

## Void

`void`可能理解为是`any`的对立面。它表示完全没有类型。你可以在没有返回值的函数类型定义中，常常看见它：

```ts
function warnUser(): void {
    alert("This is my warning message");
}
```
