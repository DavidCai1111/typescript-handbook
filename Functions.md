# 函数

函数是一切`JavaScript`应用中的最基本的代码块。在`TypeScript`中，有类和模块，但函数仍在“做实事”这方面扮演了关键的角色。`TypeScript`为传统的`JavaScript`函数增强了不少能力，使之更易用。

## 函数

和`JavaScript`一样，`TypeScript`中也可以创建具名函数或匿名函数。选择创建哪种函数完全取决于你的应用。

让我们先看看，在`JavaScript`中，它们的样子：

```js
//Named function
function add(x, y) {
    return x+y;
}

//Anonymous function
var myAdd = function(x, y) { return x+y; };
```

和在`JavaScript`中一样，一个函数可以返回在自己作用域外的变量。虽然解释这种机制超出了本文的范围，但是理解这种机制，以及使用这种技术的局限，对于理解`JavaScript`和`TypeScript`函数是至关重要的。

```js
var z = 100;

function addToZ(x, y) {
    return x+y+z;
}
```

## 函数类型

### 为函数添加类型

让我们来为先前的例子添加上类型：

```ts
function add(x: number, y: number): number {
    return x+y;
}

var myAdd = function(x: number, y: number): number { return x+y; };
```

我们可以为函数的每一个参数指定类型，包括函数的返回值。`TypeScript`会通过函数中的`return`声明来检测返回值的类型正确性。当然，你也可以不指定它。

### 书写函数类型

我们已经在一个函数上添加了类型，现在，让我们来为一个变量指定一个函数类型：

```ts
var myAdd: (x:number, y:number)=>number =
    function(x: number, y: number): number { return x+y; };
```

函数类型包含两部分：参数类型和返回值类型。当书写函数类型时，这两部分都是必须的。当我们书写参数类型时，给定的参数名仅仅是为了增强可读性，正真实现函数时不必和它们的名字一致：

```ts
var myAdd: (baseValue:number, increment:number)=>number =
    function(x: number, y: number): number { return x+y; };
```

第二部分是返回值类型。我们在参数类型和返回值类型之间使用一个胖箭头（=>）隔开了它们。正如之前提到的，这是必须的，如果函数没有返回值，你需要指定其为`void`。

另外，函数类型只能指定参数和返回值类型。外部作用域的变量是不反应在这些类型上面的。实际上，这些都是一个函数的“隐式状态”，无法反应在它的API上。

### 类型推导

在`TypeScript`中，只要你在等号一边指定了类型，编译器便会帮你去检查类型了：

```ts
// myAdd has the full function type
var myAdd = function(x: number, y: number): number { return x+y; };

// The parameters 'x' and 'y' have the type number
var myAdd: (baseValue:number, increment:number)=>number =
    function(x, y) { return x+y; };
```

这被称作“上下文类型”，是一种类型推导。它能帮助你减少代码量。

## 可选参数和默认参数

和`JavaScript`中不同，在`TypeScript`中，所有的参数都是必选的。当函数被调用时，编译器会去检查每一个参数是否的确传递了。简而言之，传入的参数数量必须和定义时一样。

```ts
function buildName(firstName: string, lastName: string) {
    return firstName + " " + lastName;
}

var result1 = buildName("Bob");  //error, too few parameters
var result2 = buildName("Bob", "Adams", "Sr.");  //error, too many parameters
var result3 = buildName("Bob", "Adams");  //ah, just right
```

在`JavaScript`中，每一个参数都是可选的，如果用户没有传递它，那它就是`undefined`。在`TypeScript`中，我们可以通过在参数后添加`?`来实现可选参数，例子：

```ts
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

var result1 = buildName("Bob");  //works correctly now
var result2 = buildName("Bob", "Adams", "Sr.");  //error, too many parameters
var result3 = buildName("Bob", "Adams");  //ah, just right
```

可选参数一定要放在必选参数的后面。如果我们想要将`firstName`变为可选，我们需要将其定义的顺序放于`lastName`之后。

在`TypeScript`中，如果用户没有传递这个参数，我们可以为其提供一个默认参数：

```ts
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

var result1 = buildName("Bob");  //works correctly now, also
var result2 = buildName("Bob", "Adams", "Sr.");  //error, too many parameters
var result3 = buildName("Bob", "Adams");  //ah, just right
```

和可选参数一样，默认参数必须在必选参数之后。

默认参数和可选参数的类型是共享的：

```ts
function buildName(firstName: string, lastName?: string) {


function buildName(firstName: string, lastName = "Smith") {
```

它们都共享类型`(firstName: string, lastName?: string)=>string`。在类型里，默认参数的默认值不见了，变成了一个可选参数标识。

## Rest参数

必选，可选，默认参数，都有一个共同点，它们都关注于一个参数。但有时，你想同时处理多个参数，或者你不能明确知道函数被调用时实际的参数数量。在`JavaScript`中，你可以在一个函数体内使用`arguments`变量得到它们。

在`TypeScript`中，你可以把这些参数全放入一个变量中：

```ts
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

var employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

Rest参数被视为是任意数量的可选参数。编译器会以数组的形式，将它们全部放入你在`...`后指定的参数中，使它们能让你在函数中使用。

类型中也可以使用rest参数：

```ts
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

var buildNameFun: (fname: string, ...rest: string[])=>string = buildName;
```

## Lambda函数，以及使用`this`

在`JavaScript`编程中，理解`this`是如何运作的是每一个程序员进阶的必经之路。`TypeScript`作为`JavaScript`的超集，`TypeScript`开发者也需要学习`this`的运作机制，并且正确地使用它。这里，我们对其做一个基本的介绍。

在`JavaScript`中，`this`是一个当函数被调用时会被设置的值。它非常的强大和灵活，通过它可以知道函数当前的执行上下文。但它往往不是很清晰了当，比如当函数作为回调函数被执行时。

例子：

```js
var deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        return function() {
            var pickedCard = Math.floor(Math.random() * 52);
            var pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

var cardPicker = deck.createCardPicker();
var pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

当我们试图运行以上代码时，我们将会得到一个异常。这是因为，在`cardPicker()`被调用时，`createCardPicker`函数所创建出的对象中的`this`会被设置为`window`，而不是`deck`对象。注意，在严格模式下，`this`将会是`undefined`，而不是`window`。

我们需要在调用返回的函数之前，将`this`绑定到正确的值上。这样，不论之后怎么调用它，`this`都只会指向`deck`对象。

为了达到这个目的，我们将使用lambda语法(()=>{})来书写函数声明。这样会自动地将`this`绑定为当前的下文环境：

```js
var deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        // Notice: the line below is now a lambda, allowing us to capture 'this' earlier
        return () => {
            var pickedCard = Math.floor(Math.random() * 52);
            var pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

var cardPicker = deck.createCardPicker();
var pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

更多关于`this`的详情，你可以参阅 [这里](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/)。

## 重载

`JavaScript`天生就是一个非常动态的语言。单个的`JavaScript`函数，可能会根据传入参数的类型不同，而返回不同类型的返回值。

```js
var suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        var pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        var pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

var myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
var pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

var pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

这里的`pickCard`函数将会根据参数类型，返回两种类型的结果。这在`TypeScript`的类型系统中，我们该如何描述它们呢？

答案是通过重载，即支持多个不同参数类型的同名函数。当编译器解析函数调用时，它自动得会去这个函数列表中选择合适的函数。例子：

```ts
var suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        var pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        var pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

var myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
var pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

var pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

有了重载，在我们调用`pickCard`时，我们也会得到类型检查。

为了使编译器能够正常地执行类型检测，它有一个非常简单的运行机制，编译器会查找此函数的重载列表，然后尝试以当前参数类型执行第一个函数，如果参数类型匹配，那么就视它为正确的重载。所以，通常来说，重载列表中的函数类型越往后越宽泛比较合适。

注意，`function pickCard(x): any`并不在函数的重载列表中，所以该函数仅有两个重载：一个接受对象参数，另一个接受数字参数。使用其他类型的参数调用`pickCard`都会得到一个报错。
