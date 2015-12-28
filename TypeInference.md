# 类型推导

这章里，我们将会讨论`TypeScript`中的类型推导。具体的说，就是哪里会出现类型推导，以及如何进行类型推导。

## 基础

在`TypeScript`中，有许多没有明确类型声明的地方，其实都可以将类型推导出来，例如：

```js
var x = 3;
```

`x`变量的类型可以推导为`number`。当初始化变量，成员，设置默认参数，或定义函数返回值类型时，这类推导就会发生。

大多数情况下，类型推导都是很直接的。在下文中，我们将探索不同推导间的细微差别。

## 最好的通用类型

当需要从一个复杂的表达式推导时，它结果的类型，是通过“最好的通用类型”原则来计算的。例子：

```js
var x = [0, 1, null];
```

为了推导出上述`x`变量的类型，我们必须考虑数组内的每一个元素。数组中有两个类型：`number`和`null`。

因为“最好的通用类型”原则，所有必须拥有它们共有的结构，但是它们之间并没有共同的超类。另一个例子：

```js
var zoo = [new Rhino(), new Elephant(), new Snake()];
```

理想情况下，我们希望`zoo`的类型为`Animal[]`，所以我们可以明确地指定这个类型，当其中没有一个元素的类型是其他两个的超集时：

```ts
var zoo: Animal[] = [new Rhino(), new Elephant(), new Snake()];
```

当没有找到合适的“最好的通用类型”时，结果类型就会是空对象，`{}`。因为这个类型没有任何成员，试图获取它的任何属性都会得到一个报错，因为所有的成员都不能有任何不明确的隐式定义。

## 上下文类型

在`TypeScript`中，还有另一个方向的类型推导，它被称为“上下文类型”。上下文类型在表达式所处的位置可以被隐式推导类型时发生。例子：

```js
window.onmousedown = function(mouseEvent) {
    console.log(mouseEvent.buton);  //<- Error  
};
```

上述的代码理论上会得到了一个类型错误。但当`TypeScript`使用`Window.onmousedown`函数的类型，来推导等号右边的类型时，它就有可能能够推导出`mouseEvent`参数的类型，并且如果该函数表达式没有在一个特定的上下文位置，那么`mouseEvent`将是`any`类型，不会得到报错。

如果明确地指定了参数的类型，那么上下文类型将被忽略：

```ts
window.onmousedown = function(mouseEvent: any) {
    console.log(mouseEvent.buton);  //<- Now, no error is given  
};
```

上下文类型推导在很多情况下都会进行。普遍的情况包括函数调用的参数，赋值语句的右半部分，类型断言，对象和数组的元素，以及返回值。上下文类型的推导行为也遵循“最好的通用类型”原则：

```ts
function createZoo(): Animal[] {
    return [new Rhino(), new Elephant(), new Snake()];
}
```

上面的例子里，最好的通用类型有四个候选：`Animal`，`Rhino`，`Elephant`和`Snake`。这些之中，`Animal`会成为“最好的通用类型”原则下最后的胜者。
